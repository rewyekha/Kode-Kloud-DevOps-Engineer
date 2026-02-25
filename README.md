# Day 1 - AWS SSH Key Creation

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a key pair with the following requirements:

* Name of the `key pair` should be `nautilus-kp`.<br>
* Key pair `type` must be `rsa`



### ✅ **Option 1 — Create Key Pair Using AWS CLI**

1. **Configure AWS CLI**\
   Make sure the AWS CLI is installed on your system and configured with the temporary credentials you noted (from the `showcreds` command on the `aws-client` host) for the **us-east-1** region (or the region you are using).
2.  **Run the Create Key Pair Command**\
    Use this command to generate the key pair _and save the private key to a local file_:

    ```bash
    aws ec2 create-key-pair \
      --key-name nautilus-kp \
      --key-type rsa \
      --query "KeyMaterial" \
      --output text > nautilus-kp.pem
    ```

    * `--key-name nautilus-kp` sets the key pair’s name.
    * `--key-type rsa` creates an RSA key pair as required.
    * `--query "KeyMaterial"` ensures only the private key content is output.
    * `> nautilus-kp.pem` saves the private key into a file named `nautilus-kp.pem`.([AWS Documentation](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html?utm_source=chatgpt.com))
3.  **Secure the Private Key File**\
    After saving the `.pem` file, set permissions so only you can read it (important for SSH):

    ```bash
    chmod 400 nautilus-kp.pem
    ```

    This prevents the private key from being world-readable.
4.  **Verify Key Pair Exists** _(optional)_

    ```bash
    aws ec2 describe-key-pairs --key-names nautilus-kp
    ```

    This should show the key pair with its name and fingerprint.

***

### ✅ **Option 2 — Create Key Pair via AWS Management Console**

If you prefer a graphical approach:

1. Sign in to the AWS console using the temporary access details you have.
2. Open **EC2 Dashboard > Network & Security > Key Pairs**.
3. Click **Create key pair**.
4. Enter:
   * **Key pair name:** `nautilus-kp`
   * **Key type:** `RSA` (the default)
5.  Click **Create key pair**.\
    The private key file will download automatically (usually named `nautilus-kp.pem`).\
    Save it securely.

    👉 The console will only provide the private key once — AWS doesn’t keep the private key in their system, so _store it safely_.([AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html?utm_source=chatgpt.com))

***

### 📌 Important Notes

* **Region-specific:** Key pairs are _specific to an AWS region_. Make sure you create it in the region you intend to launch EC2 instances (e.g., us-east-1).
* **Private Key is Vital:** The private key (`.pem`) file will be needed to SSH into EC2 instances using this key pair.
* **Security:** Always protect private key files and avoid sharing them publicly.



Kode Cloud CLI:\
![](<.gitbook/assets/image (1) (1) (1) (1).png>)

