# Simplifying Applications with Service Objects 🚆✨
This module will introduce you to the concept of Service Objects, a design pattern that streamlines your Rails applications by separating concerns, enhancing testability, and promoting cleaner code. Let's embark on this adventure to elevate your Rails applications to new heights of clarity and maintainability.

### Embracing the Principles of Service-Oriented Design
As applications grow in complexity, models and controllers can quickly become overloaded with business logic. This is where Service Objects come in, acting as a breath of fresh air, allowing us to encapsulate specific business logic or functionality into dedicated, reusable classes. A Service Object in Rails is a plain Ruby object designed to execute one single action in your business domain. Think of them as workers who have one job to do, and they do it well. This single-responsibility approach makes your application easier to understand, test, and maintain.

### Example: Processing an Order 🛍️
Imagine we're building a feature for an e-commerce platform where we need to process user orders. Instead of cluttering our models or controllers with the intricate details of this process, we'll create a Service Object to handle it elegantly.

1. Define the `OrderProcessor` Class
First, we create a Ruby class dedicated to order processing. This class will live in the `app/services` directory, a place we typically store our Service Objects in a Rails application.

```ruby
# app/services/order_processor.rb
class OrderProcessor
  def initialize(user, order_details)
    @user = user
    @order_details = order_details
  end

  def call
    validate_order_details
    create_order
    notify_user
  end

  private

  def validate_order_details
    # Implement validation logic here
  end

  def create_order
    # Logic to create the order
  end

  def notify_user
    # Send an order confirmation notification to the user
  end
end
```

2. Utilize the Service Object in a Controller
Now, let's see how we can use this Service Object within our `OrdersController` to process a new order.

```ruby
# app/controllers/orders_controller.rb
class OrdersController < ApplicationController
  def create
    order_processor = OrderProcessor.new(current_user, order_params)
    if order_processor.call
      redirect_to success_path, notice: 'Order processed successfully!'
    else
      redirect_to checkout_path, alert: 'There was an issue processing your order.'
    end
  end

  private

  def order_params
    # Strong parameter implementation
  end
end
```

## Example: Fetching Weather Data
Imagine we're building a feature for a travel planning application that provides users with real-time weather information for their destination. Instead of cluttering our models or controllers with the intricate details of this process, we'll create a Service Object to handle it elegantly. To achieve this, we'll consume an external weather API and present the data to our users.

1. Define the `WeatherService` Object
We start by creating a Ruby class dedicated to fetching weather data. This class encapsulates all the logic needed to interact with the weather API.

```ruby
# app/services/weather_service.rb
require 'http'
require 'json'

class WeatherService
  API_ENDPOINT = "https://api.pirateweather.net/forecast/"

  def initialize(location, api_key)
    @location = location # "41.8887,-87.6355"
    @api_key = api_key
  end

  def call
    response = HTTP.get("#{API_ENDPOINT}/#{@api_key}/#{@location}")
    format_response(JSON.parse(response))
  rescue StandardError => e
    Rails.logger.error("WeatherService Error: #{e.message}")
  end

  private

  def format_response(response)
    currently_hash = response.fetch("currently")

    {
      summary: currently_hash.fetch("summary"),
      temperature: currently_hash.fetch("temperature")
    }
  end
end
```
2. Utilize the Service Object in a Controller
Here's how you might use the `WeatherService` in a controller to fetch weather data for a given location and pass it to the view.

```ruby
# app/controllers/weather_controller.rb
class WeatherController < ApplicationController
  def show
    @weather_info = WeatherService.new(params[:location], ENV["PIRATE_WEATHER_API_KEY"]).call
    if @weather_info
      render :show
    else
      redirect_to root_path, alert: 'Failed to fetch weather information.'
    end
  end
end
```

### Key Considerations
- **API Key Management**: Always keep your API keys secure. Use environment variables or encrypted credentials to manage them in Rails.
- **Error Handling**: The handle_error method demonstrates basic error handling by logging the error and potentially returning a user-friendly message. Adjust this based on your application's requirements.
- **Response Formatting**: The `format_response` method is crucial for parsing and structuring the API response in a way that's useful for your application. This is where you can extract and format the data you need.
- **Caching**: To reduce API calls and improve performance, consider caching the weather data for a period of time.
- **Asynchronous Requests**: For non-critical or background tasks, consider making API calls asynchronously to improve user experience.
- **Testing**: Ensure to write tests for your Service Object, mocking external API calls to test your logic without hitting the actual API during tests.
- **Naming Conventions**: Name your Service Objects after the action they perform, such as `WeatherService` or `OrderProcessor`, to keep their purpose clear and concise.

## Embracing External APIs with Service Objects
Integrating with external APIs is a common requirement for modern web applications. Service Objects offer a clean, maintainable way to encapsulate this functionality, keeping your code organized and testable. By following the principles outlined in this example, you can extend your application's capabilities in a scalable and efficient manner, ensuring your codebase remains robust and easy to understand.

<!-- TODO: non-external api example. maybe a creator or worker service object -->

## Benefits of Using Service Objects
- **Single Responsibility Principle**: Each Service Object is responsible for one particular piece of business logic, making them easy to understand, test, and reuse.
- **Reduced Complexity**: By moving business logic out of models and controllers, Service Objects help keep these components clean and focused on their primary responsibilities.
- **Enhanced Testability**: Service Objects can be tested in isolation, allowing for more comprehensive and faster unit tests.

## Quiz
- **Question 1: What is the primary role of a Service Object in Rails applications?**
- To manage database migrations
  - Incorrect. Service Objects are not used for database management but for encapsulating business logic.
- To encapsulate specific business logic in a reusable class
  - Correct! Service Objects are designed to encapsulate and execute a specific piece of business logic, promoting cleaner and more maintainable code.
- To serve as the primary interface for the Rails application
  - Incorrect. Service Objects are not the primary interface; they are tools for managing specific tasks within the application.
- To handle all user authentication tasks
  - Incorrect. While Service Objects can be used for specific authentication tasks, their use is not limited to this area.
{: .choose_best #service_object_role title="Role of Service Objects" points="1" answer="[2]" }

- **Question 2: Which directory is commonly used to store Service Objects in a Rails application?**
- `app/models`
  - Incorrect. Models are typically used for database interaction and business rules.
- `app/controllers`
  - Incorrect. Controllers handle the request-response cycle, not specific business logic tasks.
- `app/services`
  - Correct! The `app/services` directory is the conventional place to store Service Objects in a Rails application.
- `app/helpers`
  - Incorrect. Helpers are mainly used for view-related methods.
{: .choose_best #service_object_directory title="Directory for Service Objects" points="1" answer="[3]" }

- **Question 3: When should you consider using a Service Object in a Rails application?**
- When you need to handle background jobs
  - Incorrect. Background jobs are typically handled by job processors like Sidekiq or Resque, not directly by Service Objects.
- When a model or controller begins to handle too much business logic
  - Correct, but there's more to it.
- When you need to interact with external APIs
  - Correct, but there's more to it.
- All of the above
  - Correct! Service Objects are ideal for these scenarios to maintain clean and maintainable code architecture by isolating specific business processes or tasks.
{: .choose_best #service_object_usage title="When to Use Service Objects" points="1" answer="[4]" }

- **Question 4: What is a key benefit of using Service Objects in terms of code maintenance?**
- They allow for tighter coupling of code components
  - Incorrect. Service Objects actually promote looser coupling by isolating business logic from other parts of the application.
- They increase the complexity of the codebase
  - Incorrect. The aim of Service Objects is to reduce complexity by compartmentalizing responsibilities.
- They promote code reuse and maintainability
  - Correct! By encapsulating business logic, Service Objects make the code easier to manage and reuse.
- They replace the need for models and controllers
  - Incorrect. Service Objects complement models and controllers by taking on specific tasks that don't fit neatly into the traditional roles of these components.
{: .choose_best #service_object_benefit title="Key Benefit of Service Objects" points="1" answer="[3]" }

- **Question 5: How does a Service Object enhance testability in a Rails application?**
- By requiring fewer tests due to simpler code
  - Incorrect. While simpler code is a result, the key is the isolation of business logic which directly affects testability.
- By allowing business logic to be tested in isolation from controllers and models
  - Correct! Isolating business logic in Service Objects allows for more focused and less complex tests.
- By integrating directly with the Rails testing framework
  - Incorrect. Service Objects do not inherently integrate with testing frameworks; they are simply easier to test due to their design.
- By using mocks and stubs exclusively in tests
  - Incorrect. While mocks and stubs are useful, the key benefit is the isolation of logic, not the exclusive use of these tools.
{: .choose_best #service_object_testability title="Enhancing Testability with Service Objects" points="1" answer="[2]" }

## Resources
- [Rails Service Objects: A Comprehensive Guide](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial)
- [Service Objects: Ruby On Rails Design Patterns](https://medium.com/nyc-ruby-on-rails/design-patterns-in-ruby-on-rails-service-objects-a90bf9178689)
