---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-associations"
---

# Testing Associations

Testing associations ensures your models relate to each other correctly.

## Testing belongs_to

```ruby
class CommentTest < ActiveSupport::TestCase
  test 'comment belongs to post' do
    comment = comments(:first)
    
    assert_respond_to comment, :post
    assert_instance_of Post, comment.post
  end
  
  test 'comment requires post' do
    comment = Comment.new(body: 'Test')
    
    assert_not comment.valid?
    assert_includes comment.errors[:post], 'must exist'
  end
end
```

## Testing has_many

```ruby
class PostTest < ActiveSupport::TestCase
  test 'post has many comments' do
    post = posts(:first)
    
    assert_respond_to post, :comments
    assert_kind_of ActiveRecord::Associations::CollectionProxy,
                   post.comments
  end
  
  test 'destroying post destroys comments' do
    post = posts(:first)
    comment_ids = post.comment_ids
    
    assert_difference 'Comment.count', -comment_ids.size do
      post.destroy
    end
    
    comment_ids.each do |id|
      assert_nil Comment.find_by(id: id)
    end
  end
end
```

## Testing has_many :through

```ruby
class DoctorTest < ActiveSupport::TestCase
  test 'doctor has patients through appointments' do
    doctor = doctors(:smith)
    
    assert_respond_to doctor, :patients
    assert_includes doctor.patients, patients(:john)
  end
  
  test 'can add patient via appointment' do
    doctor = doctors(:smith)
    new_patient = Patient.create!(name: 'New Patient')
    
    assert_difference 'doctor.patients.count', 1 do
      doctor.appointments.create!(patient: new_patient, date: Date.today)
    end
  end
end
```

## Testing Polymorphic Associations

```ruby
class CommentTest < ActiveSupport::TestCase
  test 'comment can belong to different commentable types' do
    post_comment = comments(:on_post)
    photo_comment = comments(:on_photo)
    
    assert_instance_of Post, post_comment.commentable
    assert_instance_of Photo, photo_comment.commentable
  end
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*