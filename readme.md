view
```ruby
#dropzone-uploadk.dropzone
  .fallback  
    - @photo = Photo.new
    = simple_form_for(@photo, url: photo_kit_inventories_path, html: { multipart: true, class: "dropzone", id: "new_upload"}) do |f|
      p.dz-message You can drop an image here or click to upload.  
      = f.input :picture, as: :hidden
      = f.input :temporary_id, input_html:{value: @kit_inventory.temporary_id},as: :hidden
      
input.dz-remove type="hidden"
javascript:
  $(document).ready(function(){
  // disable auto discover
  Dropzone.autoDiscover = false;

  // grap our upload form by its id
  $("#new_upload").dropzone({
    // restrict image size to a maximum 1MB
    maxFilesize: 1,
    maxFiles: 1,
    // changed the passed param to one accepted by
    // our rails app
    paramName: "upload[image]",
    // show remove links on each image upload
    addRemoveLinks: true,
    success: function(data, response){
        console.log("sucess yeeyeyeyey")
        console.log(response)
        $(".dz-remove").val(response.id)
    },
    //when the remove button is clicked
    removedfile: function(file){
      // grap the id of the uploaded file we set earlier
      var id = $('.dz-remove').val(); 

      // make a DELETE ajax request to delete the file
      $.ajax({
        type: 'DELETE',
        url: '/manage/kits/inventories/delete_photo?id=' + id,
        success: function(data){
          console.log(data.message);
        }
      }); 
      var _ref;
      return (_ref = file.previewElement) != null ? _ref.parentNode.removeChild(file.previewElement) : void 0;
      }
    }); 
  });
```  

parent model
```ruby
  has_many :photos, as: :photoable, dependent: :destroy
    accepts_nested_attributes_for :photos, allow_destroy: true

  attr_accessor :picture_file, :temporary_id
  after_save :update_photo_id
  def update_photo_id
     photos = Photo.where "temporary_id = '#{self.temporary_id}'"
     photos.update_all photoable_id: self.id
  end
```

migration kena tambah
```ruby
add_column :photos , temporary_id , :string
```
controller
```ruby
  def new
    @kit_inventory = Kit::Inventory.new
    @kit_inventory.temporary_id ="kit"+ Random.new.rand(1000..3000).to_s
    respond_with(@kit_inventory) 
  end
  
  def photo

    @photo = Photo.new(picture: params[:upload][:image])
    @photo.temporary_id = params[:photo][:temporary_id]
    @photo.photoable_type = "Kit::Inventory"
    logger.debug "OOO #{@photo.temporary_id}"
    @photo.save
    render json: @photo
  end

  def delete_photo
    @photo = Photo.find params[:id]
    @photo.destroy
    render json: true
  end
```

route
```ruby
        collection do
          post :photo
          delete :delete_photo
        end
```
model photos
```ruby
class Photo < ApplicationRecord
  belongs_to :photoable, polymorphic: true
  has_attached_file :picture
  validates_attachment :picture, content_type: { content_type: [ "image/jpeg", "image/jpg", "image/png"]}
end
```

gemfile
```ruby
gem 'paperclip'
gem 'dropzonejs-rails'
```
