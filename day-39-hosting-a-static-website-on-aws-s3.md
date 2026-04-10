# Day 39: Hosting a Static Website on AWS S3

The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on AWS using an S3 bucket. The S3 bucket must be configured for public access to allow external users to access the static website directly via the S3 website URL.

Task Requirements:

1. Create an S3 bucket named `xfusion-web-30228`.
2. Configure the S3 bucket for static website hosting with `index.html` as the index document.
3. Allow public access to the bucket so that the website is publicly accessible.
4. Upload the `index.html` file from the `/root/` directory of the AWS client host to the S3 bucket.
5. Verify that the website is accessible directly through the S3 website URL.

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)







<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

