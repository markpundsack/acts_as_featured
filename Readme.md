# acts_as_featured
acts_as_featured makes adding and administering feature flags easier. Feature flags allow you to separate deployment of code from delivery of features. For example you can add a set of features into your code, wrapped in a feature flag, and roll that code out without it affecting any of your users. Then you can selectively enable that feature for some of your users, validate the feature, then enable it for everyone else. Finally, remove the feature flag entirely.

It is designed for Rails, but should work for any Ruby app with a database using ActiveRecord. 

Additionally, it is designed to work with the Heroku Add-on Feature Flags as a Service (FFaaS). If the environment variable FEATURE_FLAGS_URL exists, it will use that to connect to the FFaaS service.

## Installation

In `Gemfile`:  
`gem "acts_as_featured"`

In your application root, run:  
`$ bundle install`

Generate the migration:

* If installing without a previous version or you do not mind overwriting your feature_flags table:

`$ rails g acts_as_featured:install`

* If upgrading from a previous version:

`$ rails g acts_as_featured:upgrade`

After running one o f the generators:  
`$ rake db:migrate`

## Usage
Declare _acts_as_featured_ on one of your models (usually user, account, or other object):

    class User < ActiveRecord::Base
      acts_as_featured
    end
    

Enable a feature:

    user.enable_feature(:feature1)
    
Check if a feature is enabled:

    user.feature_enabled?(:feature1)
    
Disable a feature:

    user.disable_feature(:feature1)
    
See what features are enabled for a user:

    user.features # => [:feature1]

Optionally, add or remove tags to users to enable groups of features:

    user.tag(:tag1)
    
    user.untag(:tag1)

## Administration

    User.enable_features_for_everyone(:feature1, :feature2)
    User.features_for_everyone # => [:feature1, :feature2]
    User.enable_features_for_tag(:tag1, :feature2)
    User.features_for_tag(:tag1) # => [:feature2]
    User.enable_features_for_new(:feature3)
    User.features_for_new # => [:feature3]
    User.enable_features_for_every_other_new(:feature4)
    User.features_for_every_other_new # => [:feature4]
    
Of course, the real value is when you use FFaaS and use the admin panel to manage feature flags without any code.

## Example

    class User
      acts_as_featured
      has_many :apps
    end
    
    class App
      belongs_to :user
      acts_as_featured :override_with => :user
    end
    
    User.enable_features_for_everyone(:biggerfasterstronger)
    User.enable_features_for_tag(:internal, :moreplus)
    u = User.create!
    u.features # => [:biggerfasterstronger]
    u.tag(:internal)
    u.features # => [:biggerfasterstronger, :moreplus]
    u.disable_feature(:biggerfasterstronger)
    # Disabling features doesn't override the features for everyone or tags
    u.features # => [:biggerfasterstronger,:moreplus]
    u.feature_enabled?(:moreplus) # => true
    u.feature_enabled?(:biggerfasterstronger) # => true
    a = u.apps.first
    # app checks it's user to look for overrides
    a.feature_enabled?(:biggerfasterstronger) # => true
    
    FeatureFlags.create!(:class => :User, :parent_id => u.id, :feature => :lessismore)
    FeatureFlags.create!(:class => :App, :parent_id => u.apps.first.id, :feature => :lessismore)
    ff = FeatureFlags.find_by_class_and_parent_id(:class => :User, :parent_id => u.id) # [:biggerfasterstronger,:moreplus]

## Questions
Is it better to have for_everyone set it once for all current parents, but not future ones, and rely on for_new to handle that? Or have for_everyone be a meta declaration? The latter allows for a sparse database where the records for an individual user can be blank, yet several feature flags can be associated with that account. The downside is that it's impossible to handle everyone-except-these. If that's important, a third state could be added to feature flags. e.g. on, off, disabled - where off would get the for_everyone defaults, but disabled would override that. I don't think the complexity is worth it. The value of the feature flags model is it's simplicity. Having "everyone" as a meta flag allows the data to be stored outside of the User table, or even outside of the application space altogether. e.g. a Heroku add-on doesn't need to be able to query what all the user IDs are. It just tracks the ones with exceptions. Without that, there would have to be a callback; either http or a rake task callable via `heroku run`