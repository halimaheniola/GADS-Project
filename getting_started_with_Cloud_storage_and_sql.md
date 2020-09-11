# Task: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives

In this lab, you learn how to perform the following tasks:

- Create a Cloud Storage bucket and place an image into it.

- Create a Cloud SQL instance and configure it.

- Connect to the Cloud SQL instance from a web server.

- Use the image in the Cloud Storage bucket on a web page

## Steps

1. Deploy a web server VM instance:

        gcloud compute instances create bloghost --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"--metadata-from-file startup-script=apt-get update apt-get install apache2 php php-mysql -y service apache2 restart

        gcloud compute firewall-rules create allow-http --action=ALLOW --destination=INGRESS --rules=http:80 --target-tags=http

- copy the bloghost VM instance's internal and external IP addresses to a text editor for use later in this lab.

        gcloud compute instances list --zone us-central1-a

2.Create a Cloud Storage bucket using the gsutil command line.

- Start command-line. For convenience, enter your chosen location into an environment variable called LOCATION. Enter one of these commands:

        export LOCATION=US  OR

        export LOCATION=EU  OR

		export LOCATION=ASIA

- In Cloud Shell, the DEVSHELL_PROJECT_ID environment variable contains your project ID. Enter this command to make a bucket named after your project ID:

		gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID

- Retrieve a banner image from a publicly accessible Cloud Storage location:

		gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

- Copy the banner image to your newly created Cloud Storage bucket:

		gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

- Modify the Access Control List of the object you just created so that it is readable by everyone:

		gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

3.Create the Cloud SQL instance.

- Create sql instance

		gcloud sql instances create  blog-db --password="allowaccess" --sql-version="MYSQL_5_7" --zone=us-central1-a

- copy the Public IP address for your SQL instance to a text editor for later use

		  gcloud compute sql instances list --zone us-central1-a

- Create the database instance:

		gcloud sql database create host-db --instance="blog-db"

- Create the user:

	gcloud sql users create blogdbuser --instance=blog-db-i blog-db --host='%' --password='password'

- Add a network using the following command, set the newtork by typing the external IP address of your bloghost VM instance, followed by /32:

		gcloud sql users add network web front end --network=35.192.208.2/32

4.Configure an application in a Compute Engine instance to use Cloud SQL:

- ssh into the compute instance you created earlier:

		gcloud compute ssh bloghost --zone us-central1-a

- In your ssh session on bloghost, change your working directory to the document root of the web server:

		cd/var/www/html

- Use the nano text editor to edit a file called index.php:

		sudo nano index.php

- Paste the following content into the file:

		<html>
		<head><title>Welcome to my excellent blog</title></head>
		<body>
		<h1>Welcome to my excellent blog</h1>
		<?php
		$dbserver = "CLOUDSQLIP";
		$dbuser = "blogdbuser";
		$dbpassword = "DBPASSWORD";
		// In a production blog, we would not store the MySQL
		// password in the document root. Instead, we would store it in a
		// configuration file elsewhere on the web server VM instance.

		$conn = new mysqli($dbserver, $dbuser, $dbpassword);

		if (mysqli_connect_error()) {
       		 	echo ("Database connection failed: " . 
		mysqli_connect_error());
		} else {
        		echo ("Database connection succeeded.");
		}
		?>
		</body></html>

- Press Ctrl+O, and then press Enter to save your edited file. Press Ctrl+X to exit the nano text editor. Then restart the web server:

		sudo service apache2 restart

- Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php. The URL will look like this:

		35.192.208.2/index.php

     Result: When you load the page, you will see that its content includes an error message beginning with the words:

		Database connection failed: ...

- Return to your ssh session on bloghost. Use the nano text editor to edit index.php again

		sudo nano index.php

- Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, the following message appears:

		Database connection succeeded.

5.Configure an application in a Compute Engine instance to use a Cloud Storage object.

- Enter this command to set your working directory to the document root of the web server

		cd /var/www/html

- Use the nano text editor to edit index.php

		sudo nano index.php

- Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line. Then Paste this HTML markup immediately before the URL:

		<img src='

- Place a closing single quotation mark and a closing angle bracket at the end of the URL

		'>

    Result: <img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>

- The effect of these steps is to place the line containing <img src='...'> immediately before the line containing <h1>...</h1>. Do not copy the URL shown here. Instead, copy the URL shown by the Storage browser in your own Cloud Platform project. Press Ctrl+O, and then press Enter to save your edited file. Press Ctrl+X to exit the nano text editor.

- Restart the web server

		sudo service apache2 restart

- Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.
