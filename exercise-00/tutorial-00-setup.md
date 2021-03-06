Exercise 0 - Setting things up
===========

## Objective

The objective of this exercise is to setup your custom Adobe I/O Runtime namespace. 

## Assumptions

1. You already have an Adobe I/O Runtime namespace.

2. You already have an auth code for your namespace. 

3. You already have a Magento instance and the corresponding connection details.

4. You already have [OpenWhisk CLI](https://github.com/apache/incubator-openwhisk-cli/releases) downloaded and installed.

   MacOS [PATH Variable](https://www.architectryan.com/2012/10/02/add-to-the-path-on-mac-os-x-mountain-lion/) configured to run `wsk` command

   Windows [PATH Variable](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/) configured to run `wsk` command

5. You already have an AEM instance setup and running. 

6. You already have [NodeJS](https://nodejs.org/en/download/) and NPM installed. 

   ```shell
   node -v
   ```
   
7. Git installed.  Recommend [GitHub Desktop](https://desktop.github.com/) for new Git users.

## Tasks

1. Clone the [CIF Extension Sample](https://github.com/jhuangsoftware/adobe-cif-extension-sample) repository
	
	```shell
	git clone https://github.com/jhuangsoftware/adobe-cif-extension-sample.git
	```

2. Navigate into the newly cloned Git repo, switch to the `magento` branch. 

	```shell
	cd adobe-cif-extension-sample
	git checkout magento 
	```

3. Navigate one directory structure up, clone the [Magento CIF Repository](https://github.com/adobe/commerce-cif-magento) 

	```shell
	cd ..
	git clone https://github.com/adobe/commerce-cif-magento.git
	```

4. Setup wskprops file 

	```shell
	wsk property set --apihost adobeioruntime.net --auth <Your auth code> --namespace <Your namespace>
	```

5. In the cloned `commerce-cif-magento` directory, go to the `customer-deployment` directory.

6. Copy the `credentials-example.json` file to `credentials.json` file.

7. Update the `credentials.json` file with your Magento connection details.
	```json
	{
	    "MAGENTO_SCHEMA": "http",
	    "MAGENTO_HOST": "xxxxxxx",
	    "MAGENTO_API_VERSION": "V1",
	    "MAGENTO_CUSTOMER_TOKEN_EXPIRATION_TIME": "3600",
	    "MAGENTO_AUTH_ADMIN_TOKEN": "xxxxxx",
	    "MAGENTO_MEDIA_PATH": "media/catalog/product",
	    "PRODUCT_ATTRIBUTES": ["color", "size"],
	    "GRAPHQL_PRODUCT_ATTRIBUTES": ["color", "size"],
	    "MAGENTO_IGNORE_CATEGORIES_WITH_LEVEL_LOWER_THAN": 2
	}
	```

	`MAGENTO_CUSTOMER_TOKEN_EXPIRATION_TIME` - logged in user token expiration for Magento
	
	`MAGENTO_MEDIA_PATH` - local directory where the product assets are stored in Magento
	
	`PRODUCT_ATTRIBUTES` - attributes that are relevant to determine the variants, used for display 
	
	`GRAPHQL_PRODUCT_ATTRIBUTES` - same as above, used for queries
	
	`MAGENTO_IGNORE_CATEGORIES_WITH_LEVEL_LOWER_THAN` - category tree level for navigation

8. Update `bindings-namespace` and `customer-namespace` properties in `package.json` file.

   ```json
   "customer-namespace": "YOUR_NAMESPACE",
   "bindings-namespace": "ccif-core-library",
   ```

9. From the `customer-deployment` directory, run

   ```shell
   npm install; npm run deploy
   ```

10. Confirm that the bindings were successful. 

    ```
    https://adobeioruntime.net/api/v1/web/YOUR_NAMESPACE/magento/searchProducts.http?text=jacket
    ``` 

    Complete details are available [here](https://github.com/adobe/commerce-cif-magento/tree/master/customer-deployment)

11. Optional: In the `serverless.yml` file, you can configure the `cachetime`.

    Also possible to do using 
   
	```shell
	wsk package update commerce-cif-magento-category@latest --param cachetime 400
	```

	```shell
	commerce-cif-magento-category@latest:
	      binding: /${self:custom.bindings-namespace}/commerce-cif-magento-category@latest
	      parameters:
		cachetime: 300
		$<<: ${file(credentials.json)}
	```

