---
page_type: sample
languages:
- ruby
products:
- azure
description: "This sample explains how to manage your resources and resource groups in Data Box Edge using the Azure Ruby SDK."
urlFragment: Hybrid-Resource-Manager-Ruby-Resources-And-Groups
---

# Hybrid-Resource-Manager-Ruby-Resources-And-Groups

This sample explains how to manage your
[resources and resource groups in Data Box Edge](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-key-features#resource-groups)
using the Azure Ruby SDK.

**On this page**

- [Run this sample](#run)
- [What is example.rb doing?](#example)
    - [List resource groups](#list-groups)
    - [Create a resource group](#create-group)
    - [Update a resource group](#update-group)
    - [List resources within the group](#list-resources)
    - [Delete a resource group](#delete-group)

<a id="run"></a>
## Run this sample

1. If you don't already have it, [install Ruby and the Ruby DevKit](https://www.ruby-lang.org/en/documentation/installation/).

1. If you don't have bundler, install it.

    ```
    gem install bundler
    ```

1. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/Hybrid-Resource-Manager-Ruby-Resources-And-Groups.git
    ```

1. Install the dependencies using bundle.

    ```
    cd Hybrid-Resource-Manager-Ruby-Resources-And-Groups
    bundle install
    ```

1. 	If not available, 
    [create a subscription](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-subscribe-plan-provision-vm) 
    and save the subscription ID to be used later.  
   

1. Set the following environment variables using the information from the service principal that you created.

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export user={your username for the Data Box Edge}
    export password={your password for the Data Box Edge}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    export ARM_ENDPOINT={your Data Box Edge Resource manager url}
    ```

    > [AZURE.NOTE] On Windows, use `set` instead of `export`.

1. To target DataBox Edge environment, API-Version Profile V2017_03_09 or V2018_03_01 should be used to create the resource client.

    Example:
    ```ruby
    client = Azure::Resources::Profiles::V2018_03_01::Mgmt::Client.new(options)
    ```
    ```

1. Run the sample.

    ```
    bundle exec ruby example.rb
    ```

<a id="example"></a>
## What is example.rb doing?

The sample walks you through several resource and resource group management operations.
It starts by setting up a ResourceManagementClient object using your subscription and credentials.

```ruby
subscription_id = ENV['AZURE_SUBSCRIPTION_ID'] || '11111111-1111-1111-1111-111111111111'

# This parameter is only required for AzureStack or other soverign clouds. Pulic Azure already has these settings by default.
active_directory_settings = get_active_directory_settings(ENV['ARM_ENDPOINT'])

provider = MsRestAzure::ApplicationTokenProvider.new(
	ENV['AZURE_TENANT_ID'],
	ENV['AZURE_CLIENT_ID'],
    ENV['user'],
    ENV['password'],
	active_directory_settings
)

credentials = MsRest::TokenCredentials.new(provider)
options = {
	credentials: credentials,
	subscription_id: subscription_id,
	active_directory_settings: active_directory_settings,
	base_url: ENV['ARM_ENDPOINT']
}

client = Azure::Resources::Profiles::V2018_03_01::Mgmt::Client.new(options)

```

It also sets up a ResourceGroup object (resource_group_params) to be used as a parameter in some of the API calls.

```ruby
resource_group_params = Azure::ARM::Resources::Models::ResourceGroup.new.tap do |rg|
    rg.location = `devicelocation`
end
```

There are a couple of supporting functions (`print_item` and `print_properties`) that print a resource group and it's properties.
With that set up, the sample lists all resource groups for your subscription, it performs these operations.

<a id="list-groups"></a>
### List resource groups

List the resource groups in your subscription.

```ruby
 client.resource_groups.list.value.each{ |group| print_item(group) }
```

<a id="create-group"></a>
### Create a resource group

```ruby
client.resource_groups.create_or_update('azure-sample-group', resource_group_params)
```

<a id="update-group"></a>
### Update a resource group

The sample adds a tag to the resource group.

```ruby
resource_group_params.tags = { hello: 'world' }
client.resource_groups.create_or_update('azure-sample-group', resource_group_params)
```
<a id="list-resources"></a>
### List resources within the group

```ruby
client.resource_groups.list_resources(GROUP_NAME).value.each{ |resource| print_item(resource) }
```

<a id="delete-group"></a>
### Delete a resource group

```ruby
client.resource_groups.delete('azure-sample-group')
```
