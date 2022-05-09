# Using AWS Key Management Service (KMS) for signer key

In this setup, AWS KMS owns the private key and it is never exposed. The SGN code will call the KMS API to sign the cross-chain messages.

## Create IAM user and KMS key

First, you need to create the IAM user via one of the two ways:

  - **Using the IAM Dashboard**

    Create an IAM user eg. `sgn-signer`. Select "Programmatic access" under "Access type". Keep group and policy empty. Backup the API key and secret credential securely.

  - **Using the AWS CLI**

    ```sh
    IAM_USER=sgn-kms-0
    aws iam create-user --user-name $IAM_USER --tags Key=createdby,Value=`whoami`
    aws iam create-access-key --user-name $IAM_USER
    ```

Then, create an KMS key:

In the KMS console, create a key, choose Key type: Asymmetric, Key usage: Sign and verify, Key spec: ECC_SECG_P256K1. Set the alias to `sgn-signer`. We keep it the same as the IAM user for easier mapping / management.

(Optional) Select Key Administrators and deselect "Allow admin to delete key", ONLY select `sgn-signer` as user for the key. It is OK to not have any key administrator.

## Restrict source IP of request

Read [this doc](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-aws-ip-address) for more info.

After obtaining the IP of the EC2 machine running the node, go to KMS console, select the `sgn-signer` key and switch to policy view. Add a condition with the EC2 IP to policy "Sid": "Allow use of the key":

```json
"Condition": {
    "IpAddress": {
        "aws:SourceIp": "<node-ec2-machine-ip>/32"
    }
}
```

## Configure AWS KMS in sgn.toml

Use the following format to enable signing via KMS:

```toml
signer_keystore = "awskms:<aws-region>:alias/<key-alias-name>"
signer_passphrase = "<api-key>:<api-secret>"
```

`signer_keystore` must be in the specified format. If passphrase is left empty, the code will search automatically from env variables, `~/.aws/credentials` etc.

## Obtaining the signer address using aws-kms-tools

You will need the signer address for various operations. Download and install the `aws-kms-tools` binary:

```sh
curl -L https://github.com/celer-network/sgn-v2-networks/releases/download/v1.8.0/aws-kms-tools-v1.8.0-linux-amd64.tar.gz | tar -xz
mv aws-kms-tools $GOBIN
```

Print the Ethereum signer address:

```sh
aws-kms-tools print-address --region <aws-region> --alias <key-alias-name>
```

Take a note of the printed address.

Additionally, you can use the binary to sign Ethereum messages or send transactions:

```sh
aws-kms-tools sign-message --region <aws-region> --alias <key-name> --data "0x1234"
aws-kms-tools send-tx --region <aws-region> --alias <key-name> --destination <address> --value 1 --data "0x1234"
```

Run

```sh
aws-kms-tools --help
```

to learn more.
