Hints and Tips

## S3 Website Setup
### Step 2
Execute the following command making sure to replace `YOUR_BUCKET_NAME` with the name you used in the previous section and `YOUR_BUCKET_REGION` with the region code (e.g. us-east-2) where you created your bucket.

    aws s3 sync s3://SOURCE_BUCKET s3://TARGET_BUCKET --region TARGET_BUCKET_REGION
