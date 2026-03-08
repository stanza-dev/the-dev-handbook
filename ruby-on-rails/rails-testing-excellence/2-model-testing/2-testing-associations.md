---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-associations"
---

# Testing Model Associations

## Introduction

Associations define how models relate to each other — a post has many comments, a comment belongs to a user, a doctor has many patients through appointments. Testing associations verifies that these relationships are correctly declared and that dependent behavior (like cascading deletes) works as intended. This lesson covers testing patterns for every major association type.

## Key Concepts

- **belongs_to**: A foreign key relationship where the child record references a parent. Tests should verify the association exists and that the parent is required (or optional).
- **has_many**: A one-to-many relationship. Tests should verify the collection proxy, adding and removing children, and dependent destroy behavior.
- **has_many :through**: A many-to-many relationship via a join model. Tests should verify that records on both sides are accessible through the join.
- **Dependent Destroy**: When `dependent: :destroy` is set, deleting the parent cascades to children. Tests must verify this happens and that orphaned records do not remain.

## Real World Context

A common production bug occurs when a developer forgets to add `dependent: :destroy` to a `has_many` association. Deleting a user leaves orphaned posts, comments, and notifications in the database, causing nil reference errors across the application. An association test with `assert_difference 'Comment.count', -count` catches this immediately.

## Deep Dive

### Testing belongs_to

Verify that the association method exists and that the parent record is the correct type:

```ruby
class CommentTest < ActiveSupport::TestCase
  test 'belongs to a post' do
    comment = comments(:first)

    assert_respond_to comment, :post
    assert_instance_of Post, comment.post
  end

  test 'requires a post' do
    comment = Comment.new(body: 'Orphan comment')

    assert_not comment.valid?
    assert_includes comment.errors[:post], 'must exist'
  end
end
```

The `assert_respond_to` check verifies the association is declared. The second test verifies that Rails enforces the foreign key presence (the default for `belongs_to` since Rails 5).

### Testing has_many

Verify the collection proxy and dependent destruction:

```ruby
class PostTest < ActiveSupport::TestCase
  test 'has many comments' do
    post = posts(:first)

    assert_respond_to post, :comments
    assert_kind_of ActiveRecord::Associations::CollectionProxy,
                   post.comments
  end

  test 'destroying post destroys associated comments' do
    post = posts(:first)
    comment_count = post.comments.count

    assert comment_count > 0, 'Fixture must have comments for this test'

    assert_difference 'Comment.count', -comment_count do
      post.destroy
    end
  end
end
```

The guard assertion `assert comment_count > 0` ensures the test is meaningful. Without it, a fixture with zero comments would make the test pass vacuously.

### Testing has_many :through

Verify that the through association connects both sides correctly:

```ruby
class DoctorTest < ActiveSupport::TestCase
  test 'has patients through appointments' do
    doctor = doctors(:smith)

    assert_respond_to doctor, :patients
    assert_includes doctor.patients, patients(:john)
  end

  test 'adding patient via appointment' do
    doctor = doctors(:smith)
    new_patient = Patient.create!(name: 'New Patient')

    assert_difference 'doctor.patients.count', 1 do
      doctor.appointments.create!(
        patient: new_patient,
        date: Date.current
      )
    end
  end
end
```

### Testing Polymorphic Associations

Polymorphic associations allow a model to belong to more than one type of parent:

```ruby
class CommentTest < ActiveSupport::TestCase
  test 'can belong to different commentable types' do
    post_comment = comments(:on_post)
    photo_comment = comments(:on_photo)

    assert_instance_of Post, post_comment.commentable
    assert_instance_of Photo, photo_comment.commentable
  end
end
```

## Common Pitfalls

1. **Testing associations without fixture data** — If your fixtures do not include associated records, has_many tests pass with empty collections, proving nothing. Always ensure fixtures contain the relationships you need to test.
2. **Forgetting to test dependent destroy** — The most dangerous association bug is orphaned records. Every `has_many` with business-critical children should have a test verifying cascading deletion.

## Best Practices

1. **Use assert_difference for destructive operations** — When testing destroy cascades, `assert_difference` is clearer and more reliable than counting records manually before and after.
2. **Test both directions of a relationship** — If Post `has_many :comments` and Comment `belongs_to :post`, test both sides to ensure the association is fully wired.

## Summary

- Test `belongs_to` by verifying the association exists and the parent is required.
- Test `has_many` by checking the collection proxy and verifying `dependent: :destroy` cascades correctly.
- Test `has_many :through` by verifying both sides of the many-to-many relationship are accessible.

## Code Examples

**Testing has_many and belongs_to associations with dependent destroy verification**

```ruby
require 'test_helper'

class PostTest < ActiveSupport::TestCase
  test 'has many comments' do
    post = posts(:first)
    assert_respond_to post, :comments
    assert post.comments.count > 0
  end

  test 'destroying post destroys comments' do
    post = posts(:first)
    comment_count = post.comments.count

    assert_difference 'Comment.count', -comment_count do
      post.destroy
    end
  end
end

class CommentTest < ActiveSupport::TestCase
  test 'belongs to a post' do
    comment = comments(:first)
    assert_instance_of Post, comment.post
  end
end
```


## Resources

- [Active Record Associations](https://guides.rubyonrails.org/association_basics.html) — Complete guide to all association types and their options
- [Testing Rails Applications](https://guides.rubyonrails.org/testing.html) — Official Rails testing guide with model testing examples

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*