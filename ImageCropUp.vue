<template>
    <div :id="'dropzone-' + _uid" class="drop-zone dz-clickable" data-upload-path="/api/admin/crop" data-image-previews="id">
        <div class="dz-message">Click here to upload images to the gallery...</div>
    </div>
</template>
<style>
    body{
        background-color:#ff0000;
    }
</style>
<script>
    import event from '../../../event.js'
    import Cropper from 'cropperjs'
    import Dropzone from 'dropzone'
    export default{
        data(){
            return{
                msg:'hello vue'
            }
        },
        props: ['parentId'],
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
            }

        },
        mounted(){
            const self = this
            $("#dropzone-" + this._uid).each(function(){
                var $dzContainer = $(this);
                var targetId = $dzContainer.data("image-previews");
                var myDropzone =  new Dropzone($(this)[0], {
                    url: $(this).data('upload-path'),
                    uploadMultiple: false,
                    headers: { "X-CSRF-TOKEN": $("[name='_token']").val() },
                    clickable: true,
                    acceptedFiles: "image/*",
                    success: function (file, response)
                    {
                        if ( file.status == 'success' )
                        {
                            var $file = $(file.previewElement);
                            event.$emit('cropped-image-uploaded', {
                                parentId: self.parentId,
                                file: response.image
                            })
                            $file.addClass('dz-success').delay(1000).fadeOut(400, function () {
                                $file.remove().promise().done(function ()
                                {
                                    if ( $dzContainer.find('.dz-preview').length == 0 )
                                    {
                                        $dzContainer.removeClass('dz-started');
                                    }
                                });
                            });
                        }
                    }
                });
                var modalTemplate =
                                '<div class="crop">'+
                                    '<div class="image-container">'+
                                        '<img src="images/img-placeholder.png" alt="Cropbox" id="cropbox">' +
                                    '</div>'+
                                    '<button type="button" class="btn btn-accent crop-upload">Crop</button>'+
                                '</div>'

                myDropzone.on('thumbnail', function (file) {
                    if (file.cropped) {
                        return;
                    }
                    var cachedFilename = file.name;
                    myDropzone.removeFile(file);
                    var $cropperModal = $(modalTemplate);
                    var $uploadCrop = $cropperModal.find('.crop-upload');
                    var $img = $('<img />');
                    var reader = new FileReader();
                    var cropperTool;
                    reader.onloadend = function () {
                        $cropperModal.find('.image-container').html($img);
                        $img.attr('src', reader.result);
                        cropperTool = new Cropper($($img)[0], {
                            aspectRatio: 1/1,
                            autoCropArea: 1,
                            movable: false,
                            cropBoxResizable: true,
                            minContainerWidth: 850
                        });
                    };
                    reader.readAsDataURL(file);
                    $cropperModal.insertAfter("#dropzone-" + self._uid);
                    $uploadCrop.on('click', function() {
                        var blob = cropperTool.getCroppedCanvas().toDataURL();
                        var newFile = self.dataURItoBlob(blob);
                        newFile.cropped = true;
                        newFile.name = cachedFilename;
                        myDropzone.addFile(newFile);
                        myDropzone.processQueue();
                        $cropperModal.remove();
                    });
                });
            });
        }
    }
</script>
