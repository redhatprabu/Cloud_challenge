# Kinesis Firehose

In our first scenario, we’ll set up Kinesis Firehose to stream sample data from an EC2 instance to an S3
bucket.

### Refer : https://www.lucidchart.com/documents/view/84599038-67ad-4fdb-a49c-7174835af82b/0

## Create a Stream
  Once logged into AWS, search for Kinesis. From the Kinesis Console, click Go to the Firehose console.
  In the Firehose Console, click Create Delivery Stream.
  Under the Destination dropdown menu, select Amazon S3. We must also provide a name for the stream. In
  this lab, we’ll use the name courses.
  From the S3 bucket dropdown, select New S3 bucket. For the purposes of the lab, give it the name lacourses
followed by a random string of characters. Click Create Bucket, then click Next on the delivery
stream page.
  On the next screen, set the Buffer size to 1 and the Buffer interval to 60.
  We can leave the compression and encryption settings off for this lab.
  Under the Error Logging section, select Disable.
  In the IAM Role section, choose Firehose delivery IAM role from the dropdown menu.
  The next page will allow us to confgure the role. From the IAM Role menu, select the role that’s been
  created for the lab. From the Policy Name menu, select FirehoseDeliveryRole. Finally, click the Allow
  button.
  We’ll be returned to the Firehose Console, where we can click Next. After reviewing the confguration,
  click Create Delivery Stream.

## Start the Stream
While the stream is being created, return to the Linux Academy Hands-on Lab page and copy the IP address
for the lab server. In a terminal, connect to the server via SSH using the provided credentials.
Once connected, we’ll see that some fles have already been created for the lab:
