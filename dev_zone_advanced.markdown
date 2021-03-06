This is an article about "advanced" use of the SugarCRM Ruby gem. We've previously covered using basic gem functionality, and building a simple rails app using the gem to make a CRM portal, so you might want to take a look at those first.

Using a config file
-------------------

The SugarCRM gem can use a configuration file to save you from tediously re-entering your CRM credentials each time you use it (note: this is already the case it you're using the gem with Rails). To use a configuration file, you can add your credentials to one (or more) of

    * `/etc/sugarcrm.yaml`
    * `~/.sugarcrm.yaml` (i.e. your home directory on Linux and Mac OSX)
    * a `sugarcrm.yaml` file at the root of you Windows home directory (execute `ENV[‘USERPROFILE’]` in Ruby to see which directory should contain the file)
    * `config/sugarcrm.yaml` (will need to be copied each time you upgrade or reinstall the gem)
    * a YAML file and call `SugarCRM.load_config` followed by the absolute path to your configuration file

If there are several configuration files, they are loaded sequentially in the order above and will overwrite previous values (if present). This allows you to (e.g.) have a config file in `/etc/sugarcrm.yaml` with system-wide configuration information (such as the url where SugarCRM is located) and/or defaults. Each developer/user can then have his personal configuration file in `~/.sugarcrm.yaml` with his own username and password. A developer could also specify a different location for the SugarCRM instance (e.g. a local testing instance) in his configuration file, which will take precedence over the value in `/etc/sugarcrm.yaml`.

Your configuration should be in YAML format:

    config:
      base_url: http://127.0.0.1/sugarcrm
      username: admin
      password: letmein

An example, accompanied by instructions, can be found in the `config/sugarcrm.yaml` file. In addition, a working example used for testing can be found in `test/config_test.yaml`

Extending the gem
-----------------

If you want to extend the gem's capabilities (e.g. to add methods specific to your environment), you can either

* drop your `*.rb` files in `lib/sugarcrm/extensions/` in the gem's files (see the README in that folder)

* drop your `*.rb` files in any other folder and call `SugarCRM.extensions_folder = ` followed by the absolute path to the folder containing your extensions

Let's say we want to add a `SugarCRM::Account#recent_partner?` method that will tell us if we've added this partner in SugarCRM in the last year. We'll simply create a new file at `lib/sugarcrm/extensions/extend_account.rb` containing:

    SugarCRM::Account.class_eval do
      def recent_partner?
        Time.now - self.date_entered < 1.year
      end
    end

(Note: if you want to extend the gem in Rails 3, simply add that same file somewhere in Rails.root/config/initializers.)

Now, to find out if the first account in SugarCRM was entered less than a year ago, we can simply call

    SugarCRM::Account.first.recent_partner?

Using direct access to the SugarCRM API
---------------------------------------

As previously shown, basic SugarCRM queries can be handled with the convenience methods the gem provides. For more specific and advanced use, the gem provides direct access to the SugarCRM API through the `connection` attribute. Here's how you can search for accounts with a shipping or billing address in Los Angeles (because by default the convenience method uses AND on all conditions):

    SugarCRM.connection.get_entry_list("Accounts", "accounts.billing_address_city = 'Los Angeles' OR accounts.shipping_address_city = 'Los Angeles'")

Using simultaneous sessions
---------------------------

This gem allows you to work with several SugarCRM sessions simultaneously, which can be quite useful when reproducing limited (potentially obfuscated) datasets for testing purposes, moving records to a restructured SugarCRM instance, etc.

How do multiple sessions work? On each `SugarCRM.connect` call, a namespace is returned. This namespace can then be used just like you would use the `SugarCRM` module. Make sure you do NOT store this namespace in a reserved name (such as SugarCRM). For example:

    ServerOne = SugarCRM.connect(URL1, username1, password1)
    ServerOne::User.first
    ServerTwo = SugarCRM.connect(URL2, username2, password2)
    ServerTwo::User.first

If you have only one active session, calls to SugarCRM are delegated to the active session's namespace, like so

    ServerOne = SugarCRM.connect(URL, username, password)
    ServerOne::User.first # this call does
    SugarCRM::User.first # the exact same thing as this one

To replace your session to connect with different credentials, use

    ServerOne.reconnect(new_URL, new_username, new_password)

Then your session will be reused (SugarCRM modules will be reloaded).

To disconnect an active session:

    ServerOne.disconnect!

Gem documentation can be found at https://github.com/chicks/sugarcrm, and questions/problems regarding the gem should be reported there also for quicker resolution.