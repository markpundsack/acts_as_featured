# acts_as_featured
acts_as_featured makes adding and administering feature flags easier. Feature flags allow you to separate deployment of code from delivery of features. For example you can add a set of features into your code, wrapped in a feature flag, and roll that code out without it affecting any of your users. Then you can selectively enable that feature for some of your users, validate the feature, then enable it for everyone else. Finally, remove the feature flag entirely.

It is designed for Rails, but should work for any Ruby app with a database using ActiveRecord. Additionally, it is designed to work with the Heroku Add-on Feature Flags as a Service (FFaaS).

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

    User.set_default_features(:feature1, :feature2)
    User.default_features # => [:feature1, :feature2]
    User.set_tag_features(:tag1, :feature2)

## Example

    User.set_default_features(:biggerfasterstronger)
    User.set_tag_features(:internal, :moreplus)
    user = User.create!
    user.tag(:internal)
    user.disable_feature(:biggerfasterstronger)
    user.feautres # => [:moreplus]
    user.feature_enabled?(:moreplus) # => true
    user.feature_enabled?(:biggerfasterstronger) # => false