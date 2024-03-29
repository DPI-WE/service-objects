# Simplifying Applications with Service Objects 🚆✨
This module will introduce you to the concept of Service Objects, a design pattern that streamlines your Rails applications by separating concerns, enhancing testability, and promoting cleaner code. Let's embark on this adventure to elevate your Rails applications to new heights of clarity and maintainability.

### Embracing the Principles of Service-Oriented Design
As applications grow in complexity, models and controllers can quickly become overloaded with business logic. This is where Service Objects come in, acting as a breath of fresh air, allowing us to encapsulate specific business logic or functionality into dedicated, reusable classes. A Service Object in Rails is a plain Ruby object designed to execute one single action in your business domain. Think of them as workers who have one job to do, and they do it well. This single-responsibility approach makes your application easier to understand, test, and maintain.

<!-- TODO: add another non api client example -->

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

## Resources
- [Rails Service Objects: A Comprehensive Guide](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial)
- [Service Objects: Ruby On Rails Design Patterns](https://medium.com/nyc-ruby-on-rails/design-patterns-in-ruby-on-rails-service-objects-a90bf9178689)
