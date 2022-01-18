# kitchen-vra

A driver to allow Test Kitchen to consume vRealize Automation (vRA) resources to perform testing.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'kitchen-vra'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install kitchen-vra

Or even better, install it via ChefDK:

    $ chef gem install kitchen-vra

## Usage

After installing the gem as described above, edit your .kitchen.yml file to set the driver to 'vra' and supply your login credentials:

```yaml
driver:
  name: vra
  username: myuser@corp.local
  password: mypassword
  tenant: mytenant
  base_url: https://vra.corp.local
  verify_ssl: true
```

If you want username and password to be prompted, remove usename and password in your .kitchen.yml as shown below:

```yaml
driver:
  name: vra
  tenant: mytenant
  base_url: https://vra.corp.local
  verify_ssl: true
```
If you don't want to explicitly specify username and password in the kitchen.yml, you have an option to set it in the environment variable as 

    $ export VRA_USER_NAME='myuser@corp.local'
    $ export VRA_USER_PASSWORD='mypassword'

Then configure your platforms. You will need to specify project_id, image_mapping, flavor_mapping and version for the platform. image_mapping specifies the OS image for a machine and the flavor_mapping specifies the CPU count and RAM of the machine. Either a catalog_id or a catalog_name is required for each platform. If both catalog_id and catalog_name are mentioned in .kitchen.yml then catalog_name would be used to derive the catalog_id and this catalog_id would override the catalog_id being passed in .kitchen.yml. In the below example as can be seen we are using catalog_id for centos6 driver while catalog_name for the centos7 driver just to demonstrate that we can use either of the two.


```yaml
platforms:
  - name: centos6
    driver:
      catalog_id: e9db1084-d1c6-4c1f-8e3c-eb8f3dc574f9
      project_id: 6ba69375-79d5-42c3-a099-7d32739f71a9
      image_mapping: SQL 2016
      flavor_mapping: Small
      version: 1
  - name: centos7
    driver:
      catalog_name: my_catalog_name
      project_id: 6ba69375-79d5-42c3-a099-7d32739f71a9
      image_mapping: VRA-nc-lnx-ce8.4-Docker
      flavor_mapping: Small
      version: 1
```



Other options that you can set include:

 * **request_timeout**: amount of time, in seconds, to wait for a vRA request to complete. Default is 600 seconds.
 * **server_ready_retries**: Number of times to retry the "waiting for server to be ready" check. In some cases, this will error out immediately due to DNS propagation issues, etc. Setting this to a number greater than 0 will retry the `wait_until_ready` method with a growing sleep in between each attempt. Defaults to 1. Set to 0 to disable any retrying of the `wait_until_ready` method.
 * **subtenant_id**: the Business Group ID to list as the owner. This is required if the catalog item is a shared/global item; we are unable to determine the subtenant_id from the catalog, and vRA requires it to be set on every request.
 * **subtenant_name**: the Business Group Name as the owner. This can be passed instead of subtenant_id and would act as a more friendly name. subtenant_id would be internally retrieved based on the provided subtenant_name. In case both subtenant_id and subtenant_name are passed, subtenant_name would take the precendence and would try to retrieve subtenant_id based on subtenant_name passed.
 * **private_key_path**: path to the SSH private key to use when logging in. Defaults to '~/.ssh/id_rsa' or '~/.ssh/id_dsa', preferring the RSA key. Only applies to instances where SSH transport is used; i.e., does not apply to Windows hosts with the WinRM transport configured.
 * **use_dns**: Defaults to `false`.  Set to `true` if vRA doesn't manage vm ip addresses.  This will cause kitchen to attempt to connect via hostname.
 * **dns_suffix**: Defaults to `nil`.  Set to your domain suffix, for example 'mydomain.com'.  This only takes effect when `use_dns` == true and is appended to the hostname returned by vRA.
 * **extra_parameters**: a hash of other data to set on a catalog request, most notably custom properties. Allows updates to existing properties on the blueprint as well as the addition of new properties. Each key in the hash is the property name, and the value is a another hash containing the value data type and the value itself. It is possible to use a `~` to add nested parameters.

These settings can be set globally under the top-level `driver` section, or they can be set on each platform, which allows you to set globals and then override them. For example, this configuration would set the CPU count to 1 except on the "large" platform:

```yaml
driver:
  name: vra

platforms:
  - name: small
    driver:
      catalog_name: my_catalog_name_small
      catalog_id: 8a189191-fea6-43eb-981e-ee0fa40f8f57
      project_id: 6ba69375-79d5-42c3-a099-7d32739f71a9
      image_mapping: SQL 2016
      flavor_mapping: Small
      version: 1
      extra_parameters:
        mycustompropname:
          type: string
          value: smallvalue
        Vrm.DataCenter.Location:
          type: string
          value: Non-Prod
  - name: large
    driver:
      catalog_name: my_catalog_name_large
      catalog_id: 1d7c6122-18fa-4ed6-bd13-8a33b6c6ed50
      project_id: 6ba69375-79d5-42c3-a099-7d32739f71a9
      image_mapping: VRA-nc-lnx-ce8.4-Docker
      flavor_mapping: Large
      version: 1
      subtenant_name: my_subtenant_name
      extra_parameters:
        mycustompropname:
          type: string
          value: largevalue
        Vrm.DataCenter.Location:
          type: string
          value: Prod
```

## License and Authors

Author:: Chef Partner Engineering (<partnereng@chef.io>)

Copyright:: Copyright (c) 2015-2019 Chef Software, Inc.

License:: Apache License, Version 2.0

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the License at

```
http://www.apache.org/licenses/LICENSE-2.0
```

Unless required by applicable law or agreed to in writing, software distributed under the
License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the License for the specific language governing permissions
and limitations under the License.

## Contributing

We'd love to hear from you if this doesn't work in your vRA environment. Please log a GitHub issue, or even better, submit a Pull Request with a fix!

1. Fork it ( https://github.com/chef-partners/kitchen-vra/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
