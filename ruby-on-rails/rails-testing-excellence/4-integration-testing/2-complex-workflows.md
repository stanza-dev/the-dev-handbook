---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-complex-workflows"
---

# Complex Workflows

Complex workflows involve multiple models, controllers, and user interactions.

## E-commerce Checkout Flow

```ruby
class CheckoutFlowTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:buyer)
    @product = products(:widget)
  end
  
  test 'complete checkout process' do
    # Login
    post login_url, params: {
      email: @user.email,
      password: 'password'
    }
    
    # Add item to cart
    post cart_items_url, params: {
      product_id: @product.id,
      quantity: 2
    }
    assert_response :redirect
    follow_redirect!
    
    # View cart
    get cart_url
    assert_response :success
    assert_match @product.name, response.body
    
    # Proceed to checkout
    get checkout_url
    assert_response :success
    
    # Submit order
    assert_difference(['Order.count', 'OrderItem.count'], 1) do
      post orders_url, params: {
        order: {
          shipping_address: '123 Main St',
          payment_method: 'credit_card'
        }
      }
    end
    
    # Verify order confirmation
    follow_redirect!
    assert_match 'Order confirmed', response.body
    assert_match Order.last.confirmation_number, response.body
    
    # Cart should be empty
    get cart_url
    assert_match 'Your cart is empty', response.body
  end
end
```

## Testing Email in Workflows

```ruby
class PasswordResetFlowTest < ActionDispatch::IntegrationTest
  test 'complete password reset flow' do
    user = users(:david)
    
    # Request password reset
    assert_emails 1 do
      post password_resets_url, params: {
        email: user.email
      }
    end
    
    # Get reset token from email
    email = ActionMailer::Base.deliveries.last
    reset_token = extract_token_from_email(email)
    
    # Visit reset page
    get edit_password_reset_url(reset_token)
    assert_response :success
    
    # Submit new password
    patch password_reset_url(reset_token), params: {
      user: {
        password: 'newpassword123',
        password_confirmation: 'newpassword123'
      }
    }
    
    # Should be logged in and redirected
    follow_redirect!
    assert_match 'Password has been reset', response.body
    
    # Can login with new password
    delete logout_url
    post login_url, params: {
      email: user.email,
      password: 'newpassword123'
    }
    assert_redirected_to dashboard_url
  end
  
  private
  
  def extract_token_from_email(email)
    email.body.encoded.match(/reset_password\/(.+?)\"/)&.[](1)
  end
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*