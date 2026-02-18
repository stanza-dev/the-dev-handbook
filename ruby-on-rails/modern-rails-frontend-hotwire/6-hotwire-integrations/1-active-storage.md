---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-hotwire-active-storage"
---

# Hotwire Active Storage

Combine Active Storage direct uploads with Hotwire for great UX.

## Direct Upload Setup

```javascript
// app/javascript/application.js
import * as ActiveStorage from '@rails/activestorage'
ActiveStorage.start()
```

## Upload Controller

```javascript
// controllers/upload_controller.js
import { Controller } from '@hotwired/stimulus'
import { DirectUpload } from '@rails/activestorage'

export default class extends Controller {
  static targets = ['input', 'progress', 'preview']
  static values = { url: String }
  
  upload() {
    const file = this.inputTarget.files[0]
    if (!file) return
    
    this.showProgress()
    
    const upload = new DirectUpload(file, this.urlValue, this)
    
    upload.create((error, blob) => {
      if (error) {
        this.handleError(error)
      } else {
        this.handleSuccess(blob)
      }
    })
  }
  
  directUploadWillStoreFileWithXHR(request) {
    request.upload.addEventListener('progress', event => {
      const progress = (event.loaded / event.total) * 100
      this.updateProgress(progress)
    })
  }
  
  showProgress() {
    this.progressTarget.classList.remove('hidden')
  }
  
  updateProgress(percent) {
    this.progressTarget.style.width = `${percent}%`
  }
  
  handleSuccess(blob) {
    // Add hidden field with blob signed ID
    const hiddenField = document.createElement('input')
    hiddenField.type = 'hidden'
    hiddenField.name = this.inputTarget.name
    hiddenField.value = blob.signed_id
    this.element.appendChild(hiddenField)
    
    // Show preview
    if (blob.content_type.startsWith('image/')) {
      this.previewTarget.src = `/rails/active_storage/blobs/${blob.signed_id}/${blob.filename}`
    }
  }
}
```

## Form with Upload

```erb
<%= form_with model: @product, data: { controller: 'upload', upload_url_value: rails_direct_uploads_url } do |f| %>
  <div class="upload-area">
    <%= f.file_field :image, 
        data: { upload_target: 'input', action: 'change->upload#upload' },
        accept: 'image/*' %>
    
    <div class="progress hidden" data-upload-target="progress">
      <div class="progress-bar"></div>
    </div>
    
    <img data-upload-target="preview" class="preview">
  </div>
  
  <%= f.submit 'Save' %>
<% end %>
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*