---
layout: post
title: Usage of Service Object Pattern and Mixin in Rails MVC
---

A common scenario in your Rails project is where you have a super fat model, containing bunch of methods that get called by the controller. Suppose you are running a backend application to serve requests from clients. This application is a content management system of a food delivery service, which means this application deals with any content related business action of all food products. The table relation looks like this:

```
 ____________            ____________            ____________          
|            |          |            |          |            |          
| Restaurant |---------<|  Category  |---------<|    Menu    |
|____________|          |____________|          |____________|          


```
One restaurant can have multiple categories, and one category can have multiple menus. Table of `categories` and `menus` have boolean column `active`.

## Mixin to the rescue

Suppose there is a business feature where the restaurant owner can show/hide the category or menu to the customers, normally the implementation at their models are like this.

```
class Category < ActiveRecord::Base
	#...
	
	def activate
		self.active = true
		self.save		
	end
	
	def deactivate
		self.active = false
		self.save
	end
end
```

```
class Menu < ActiveRecord::Base
	#...
	
	def activate
		self.active = true
		self.save		
	end
	
	def deactivate
		self.active = false
		self.save
	end
end
```
As there is a common trait of both models here, a way to make this better is by having mixin, which in Rails is implemented using Concern. The implementation will look like this.

```
module Activatable
	extend ActiveSupport::Concern
	
	def activate
		self.active = true
		self.save		
	end
	
	def deactivate
		self.active = false
		self.save
	end
end
```
```
class Category < ActiveRecord::Base
	include Activatable
	#...
end
```
```
class Menu < ActiveRecord::Base
	include Activatable
	#...
end
```
Here, both models have `activate` and `deactivate` methods implemented and can be used normally but now the model classes are slightly shorter than before where they have their own implementation of `activate` and `deactivate`. While the sample above is probably a cheap duplication, it still adds value to the code if a common behavior can be shared between models.

## Mixin NOT to the rescue

Now suppose `Menu` class has hundreds of lines of code and much more lines of code in the spec, it is better to not getting tempted by moving some of the methods into modules for the sake of reducing lines of code. Example like below is not preferrable.

```
class Menu < ActiveRecord::Base
	#...
	
	def set_weight
		#...	
	end
end
```
to become

```
module Weightable
	extend ActiveSupport::Concern
	
	def set_weight
		#...	
	end
end
```
```
class Menu < ActiveRecord::Base
	include Weightable
	#...
end
```
As the behavior of being able to set the weight of a menu domain is not something to be shareable to others, it does not make sense to put it into module as it will only reduce the readability of the code. Not much added value will be acquired by doing this.

## Service Object Pattern to the rescue
Now, what if you still want to make your fat model thinner after implementing mixins as much as possible? Moving long method into a service object is probably the answer. Using above domain sample, suppose a restaurant can have an `image_url`, which is the restaurant's image shown to users. Now we currently have a long method in the restaurant which gets called by controller.

```
class Restaurant < ActiveRecord::Base
	#...
	
	def set_image(file)
		# upload the image to cloud, return false if it fails
		# generate the url to be stored
		self.image_url = uploaded_image_url
		self.save
	end
end
```
```
class Api::V1::Restaurants::ImagesController < Api::V1::BaseController
	before_action :validate_file, :set_restaurant
	
	def create
		if @restaurant.set_image(params[:file])
			render status: :ok
		else
			# render error response body
	end
	
	#...
end
```
We can pull this complex functionality into `RestaurantPhotoUploader` so `Restaurant` class does not need to have `set_image` method.

```
class RestaurantPhotoUploader
	def initialize(restaurant: restaurant, file: file)
		@restaurant = restaurant
		@file = file
	end
	
	def process
		# upload the image to cloud
		# generate the url to be stored
		restaurant.image_url = uploaded_image_url
		restaurant.save
	end
end
```
```
class Api::V1::Restaurants::ImagesController < Api::V1::BaseController
	before_action :validate_file, :set_restaurant
	
	def create
		uploader = RestaurantPhotoUploader.new(restaurant: @restaurant, file: params[:file])
		if uploader.process
			render status: :ok
		else
			# render error response body
	end
	
	#...
end
```
Or if we have a requirement to allow menus (or other domains) to have image, we can modify `RestaurantPhotoUploader` to be more generic like this.

```
class PhotoUploader
	def initialize(model: model, file: file)
		@model = model
		@file = file
	end
	
	def process
		# upload the image to cloud
		# generate the url to be stored
		model.image_url = uploaded_image_url
		model.save
	end
end
```
Example above strongly forces all domains that require photo upload feature to have column called `image_url`. To even have more flexibility, this can be tweaked by having smaller `PhotoUploader` returning a stateful `ProcessResult` like this.

```
class PhotoUploader
	def initialize(model: model, file: file)
		@model = model
		@file = file
	end
	
	def process
		# upload the image to cloud
		# generate the url to be stored
		ProcessResult.new(success: true, result: uploaded_image_url)
	end
end
```
Above will provide flexibility of usages like these.

```
restaurant.set_cover_image(process_result.result)
restaurant.set_profile_image(process_result.result)
menu.set_image_url(process_result.result)

# and so on
```

## Conclusion

Mixin (by using `ActiveSupport::Concern`) can be used to make model classes simpler by sharing common behavior. However, I personally feel that it is not necessary to be used if the intention is just to make a single fat model slimmer. A fat model will always have some methods with complex logic. It is better to move these methods into several service objects which can be tested independently.