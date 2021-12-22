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
signer_keystore = "awskms:<aws-region>:alias/<key-name>"
signer_passphrase = "<api-key>:<api-secret>"
```

`signer_keystore` must be in the specified format. If passphrase is left empty, the code will search automatically from env variables, `~/.aws/credentials` etc.

## Obtaining the signer address

You will need the signer address for various operations. Download and install the `aws-kms-test` binary:

```sh
curl -L https://github.com/celer-network/sgn-v2-networks/releases/download/v1.4.3/aws-kms-test-linux-amd64.tar.gz | tar -xz
mv aws-kms-test $GOBIN
```

Test that signing works:

```sh
aws-kms-test -r <aws-region> -k <key-name>
```

Take a note of the printed Ethereum signer address.
