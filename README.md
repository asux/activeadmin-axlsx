Active Admin Axlsx: Office Open XML Spreadsheet Export for Active Admin
====================================

This is a fork of the original [activeadmin-axlsx](https://github.com/randym/activeadmin-axlsx)

Synopsis
--------

This gem provides automatic OOXML (xlsx) downloads for Active Admin
resources. It lets you harness the full power of Axlsx when you want to
but for the most part just stays out of your way and adds a link next to
the csv download for xlsx (Excel/numbers/Libre Office/Google Docs)

![Screen 1](https://github.com/welaika/activeadmin-axlsx/raw/master/screen_capture.png)

Usage example:
Simply add the following to your Gemfile and you are good to go.
All resource index views will now include a link for download directly
to xlsx.

```
gem 'activeadmin-axlsx'
```

Cool Toys
---------

Here are a few quick examples of things you can easily tweak.
Axlsx supports A LOT of the specification so if you are looking to do
something adventurous please ping me on irc. (freenode#axlsx)

## Localize column headers

```ruby
#app/admin/posts.rb
ActiveAdmin.register Post do
  config.xlsx_builder.i18n_scope = [:active_record, :models, :posts]
end
```

## Use blocks for adding computed fields

```ruby
#app/admin/posts.rb
ActiveAdmin.register Post do
  config.xlsx_builder.column('author_name') do |resource|
    resource.author.name
  end
end
```

## Change the column header style

```ruby
#app/admin/posts.rb
ActiveAdmin.register Post do
  config.xlsx_builder.header_style = { :bg_color => 'FF0000',
                                       :fg_color => 'FF' }
end
```

## Remove columns

```ruby
#app/admin/posts.rb
ActiveAdmin.register Post do
  config.xlsx_builder.delete_columns :id, :created_at, :updated_at
end
```

# Using the DSL

Everything that you do with the config'd default builder can be done via
the resource DSL.

Below is an example of the DSL

```ruby
ActiveAdmin.register Post do

  # i18n_scope and header style are set via options
  xlsx(:i18n_scope => [:active_admin, :axlsx, :post],
       :header_style => {:bg_color => 'FF0000', :fg_color => 'FF' }) do

    # Specify that you want to white list column output.
    # whitelist

    # Do not serialize the header, only output data.
    # skip_header

    # deleting columns from the report
    delete_columns :id, :created_at, :updated_at

    # adding a column to the report
    column(:author) { |resource| "#{resource.author.first_name} #{resource.author.last_name}" }

    # creating a chart and inserting additional data with after_filter
    after_filter { |sheet|
      sheet.add_row []
      sheet.add_row ['Author Name', 'Number of Posts']
      data = []
      labels = []
      User.all.each do |user|
        data << user.posts.size
        labels << "#{user.first_name} #{user.last_name}"
        sheet.add_row [labels.last, data.last]
      end
      chart_color =  %w(88F700 279CAC B2A200 FD66A3 F20062 C8BA2B 67E6F8 DFFDB9 FFE800 B6F0F8)
      sheet.add_chart(::Axlsx::Pie3DChart, :title => "post by author") do |chart|
        chart.add_series :data => data, :labels => labels, :colors => chart_color
        chart.start_at 4, 0
        chart.end_at 7, 20
      end
    }

    # iserting data with before_filter
    before_filter do |sheet|
      sheet.add_row ['Created', Time.zone.now]
      sheet.add_row []
    end
  end
end
```

# Specs
------
Running specs for this gem requires that you construct a rails application.
To execute the specs, navigate to the gem directory,
run bundle install and run these to rake tasks:

```
bundle exec rake setup
```

```
bundle exec rake
```
# Changelog

**2017.06.20**
  - Forked and upgraded gem to work with the last ActiveAdmin, Rails 5.1.x, RubyZip

**2013.10.12**
  - Upgraded to most recent version of Axlsx. This introduces some non-backwards compatible
    changes and pushes rubyzip up to 1.0.0
  - Added support for scoped collections #18
  - Added support to specify whitelist in in configuration. This will clear all columns and
    you can then specify only the fields you want.
  - Added support for skip_header in the builder/DSL.
  - Moved initialization into after config block in an attempt to not crunch assets:precompile

**2013.06.02** Release 2.1.2
  - builder#collection is now set on serialize and is available in before and after filters.
  - Code cleanup

**2013.04.18** Release 2.1.1
  - Fixed issue with repeating data in sheets across downloads
  - Updated to use activeadmin 0.6.0+ which supports plugins.

**2013.03.21** Release 2.0.1
  - Fixed an issue with missing objects when using the DSL.
    Huge thanks to [Fivell](https://github.com/Fivell)

**2012.11.29** Release 2.0.0
  - resouce content column are now pre-populated.
  - added before and after filters
  - 100% spec coverage

**2012.11.16**
  - Fixed DSL referencing
  - Added delete_columns to builder and DSL

# Copyright and License
----------

activeadmin-axlsx &copy; 2012 ~ 2013 by [Randy Morgan](mailto:digial.ipseity@gmail.com).

activeadmin-axlsx is licensed under the MIT license. Please see the LICENSE document for more information.
