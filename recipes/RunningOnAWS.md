#Running ArangoDB on AWS

ArangoDB is available as AMI in the [AWS Marketplace][1]. 
If you've already a running ArangoDB image on AWS and need an update, please have a look at [Updating ArangoDB on AWS](UpdateArangoDBOnAWS)

Here is a quick guide how to start:

First of all go the [ArangoDB marketplace][2], select the lastest version and click on **Continue**

![AWS marketplace](assets/RunningOnAWS/marketplace.png)

Now log in with your AWS Account or create a new one.

After that you should select the **1-Click Launch** tab and select the size of the instance (**EC2 Instance Type**) you wish to use.

Now you can continue with a click on **Accept Terms & Launch with 1-Click**.

**Note**: If you do not have a key pair a warning will appear after clicking and you will be asked to generate a key pair.

![Instance Size](assets/RunningOnAWS/instance.png)

You successfully launched an ArangoDB instance on AWS.

![Launch ArangoDB](assets/RunningOnAWS/launch.png)

Now you can launch your ArangoDB instance in [your Software library][3]. Click on **Access Software** and you will see the ArangoDB Web Interface.

![Web Interface](assets/RunningOnAWS/webInterface.png)

**Author**: [Thomas Schmidts](https://github.com/13abylon)

**Tags** : #aws, #amazon, #howto

[1]: https://aws.amazon.com/de/
[2]: https://aws.amazon.com/marketplace/search/results/ref=dtl_navgno_search_box?page=1&searchTerms=arangodb
[3]: https://aws.amazon.com/marketplace/library
