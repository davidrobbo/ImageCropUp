<template>
    <div>
        <div :id="'dropzone-' + _uid" class="drop-zone dz-clickable" data-upload-path="/api/admin/crop" data-image-previews="id">
            <div class="dz-message">Click here to upload images to the gallery...</div>
        </div>
        <div class="crop" v-if="cropperActive" :id="'cropper-' + _uid">
            <div class="image-container">
                <img src="images/img-placeholder.png" alt="Cropbox" :id="'cropbox-' + _uid">
                <div class="button-holder">
                    <button type="button" class="m5 btn btn-accent crop-upload" @click="crop"><i class="fa fa-crop"></i></button>
                    <button class="m5 btn btn-primary" @click="rotate(45)"><i class="fa fa-rotate-right"></i></button>
                    <button class="m5 btn btn-primary" @click="rotate(-45)"><i class="fa fa-rotate-left"></i></button>
                    <button class="m5 btn btn-primary" @click="scaleX(-1)"><i class="fa fa-arrow-left"></i></button>
                    <button class="m5 btn btn-primary" @click="scaleX(1)"><i class="fa fa-arrow-right"></i></button>
                    <button class="m5 btn btn-primary" @click="scaleY(-1)"><i class="fa fa-arrow-up"></i></button>
                    <button class="m5 btn btn-primary" @click="scaleY(1)"><i class="fa fa-arrow-down"></i></button>
                </div>
            </div>
        </div>
    </div>
</template>
<style>
    body{
        background-color:#ff0000;
    }
    .crop {
        position: relative;
    }
    .button-holder {
        position: absolute;
        top: 5%;
        right: 5%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;,
        background-color: #ECF0F5;
        padding: 10px;
    }
    .m5 {
        margin-bottom: 5px;
    }
</style>
<script>
    import event from '../../event.js'
    import Cropper from 'cropperjs'
    import Dropzone from 'dropzone'
    export default{
        name: "ImageCropper",
        data(){
            return{
                msg:'hello vue',
                myDropzone: null,
                cropperTool: null,
                cropperActive: false,
                ratio: 1,
                url: "/api/admin/crop",
                token: document.getElementsByName("_token")[0].value,
                cachedFilename: ""
            }
        },
        props: ['parentId', 'index', 'aspectRatio'],
        components:{
        },
        methods: {
            dataURItoBlob(dataURI){
                var byteString = atob(dataURI.split(',')[1]);
                var ab = new ArrayBuffer(byteString.length);
                var ia = new Uint8Array(ab);
                for (var i = 0; i < byteString.length; i++) {
                    ia[i] = byteString.charCodeAt(i);
                }
                return new Blob([ab], { type: 'image/jpeg' });
            },
            rotate(n){
                this.cropperTool.rotate(n)
            },
            scaleX(a){
                this.cropperTool.scaleX(a)
            },
            scaleY(a){
                this.cropperTool.scaleY(a)
            },
            crop(){
                var blob = this.cropperTool.getCroppedCanvas().toDataURL();
                var newFile = this.dataURItoBlob(blob);
                newFile.cropped = true;
                newFile.name = this.cachedFilename;
                this.myDropzone.addFile(newFile);
                this.myDropzone.processQueue();
                this.cropperActive = false
            }
        },
        mounted(){
            this.$nextTick( () => {
                this.ratio = this.aspectRatio ? this.aspectRatio : this.ratio
                const self = this
                var $dzContainer = self.$el.querySelector("#dropzone-" + self._uid)
                self.myDropzone =  new Dropzone(self.$el.querySelector("#dropzone-" + self._uid), {
                    url: self.url,
                    uploadMultiple: false,
                    headers: { "X-CSRF-TOKEN": self.token },
                    clickable: true,
                    acceptedFiles: "image/*",
                    success: function (file, response)
                    {
                        if ( file.status == 'success' )
                        {
                            var $file = $(file.previewElement);
                            try {
                                event.$emit('cropped-image-uploaded', {
                                    parentId: self.parentId,
                                    index: self.index,
                                    file: response.image
                                })
                            } catch (e) {
                                self.$emit('cropped-image-uploaded', {
                                    parentId: self.parentId,
                                    index: self.index,
                                    file: response.image
                                })
                            }
                            $file.addClass('dz-success').delay(1000).fadeOut(400, function () {
                                $file.remove().promise().done(function ()
                                {
                                    if ( !$dzContainer.querySelector('.dz-preview') )
                                    {
                                        $dzContainer.classList.remove('dz-started');
                                    }
                                });
                            });
                        }
                    }
                });
                self.myDropzone.on('thumbnail', function (file) {
                    if (file.cropped) {
                        return;
                    }
                    self.cachedFilename = file.name;
                    self.myDropzone.removeFile(file);
                    var reader = new FileReader();
                    reader.onloadend = function () {
                        self.$el.querySelector("#cropbox-" + self._uid).setAttribute('src', reader.result);
                        self.cropperTool = new Cropper(self.$el.querySelector("#cropbox-" + self._uid), {
                            aspectRatio: self.ratio,
                            autoCropArea: 1,
                            movable: false,
                            cropBoxResizable: true,
                            minContainerWidth: 850,
                            rotatable: true,
                            scalable: true
                        });
                    };
                    reader.readAsDataURL(file);
                    self.cropperActive = true
                });
            })
        },
        beforeDestroy(){
            this.myDropzone.destroy()
        }
    }
</script>
