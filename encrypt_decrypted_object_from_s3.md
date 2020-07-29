# Encrypt - decrypted object from s3 bucket using python


AWS allows you to assume roles in other AWS accounts. Its a nice feature that allows you to log into 1 account, assume a role in another account, and issue API commands as if you had signed into the 2nd account. You can have all users sign into 1 central account, then assume roles into other accounts based on a job role. Using the AWS gui, this is a few mouse clicks, but here I’ll show you how to assume a role using BOTO3.


I have access keys for my base account and assume-role for dev,qa or prod to access the bucket data
I assume you have appropritate access to decrypt the data and not focusing on equired access 

**Aim : Read the s3 bucket data using assume role and decrypt&print the object in readable format.**


### Import required modules boto3 and cryptography 

``` pip3 install boto3 cryptography```

### Update following parameters

    RoleArn="<role-arn>" eg: arn:aws:iam::123412341234:role/rolename
    RoleSessionName='<session-name>' eg:koti
    bucket='<bucket-name>' eg:bucket-pjgxnybip8abl
    key='<bucket-key-path>' eg: path/object

```
import boto3
import json
import base64
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

def get_data_from_s3(bucket, key):
    kms = boto3.client(
        'kms',
        aws_access_key_id=newsession_id,
        aws_secret_access_key=newsession_key,
        aws_session_token=newsession_token
    )
    s3 = boto3.client(
        's3',
        region_name='eu-west-1',
        aws_access_key_id=newsession_id,
        aws_secret_access_key=newsession_key,
        aws_session_token=newsession_token
    )
    obj = s3.get_object(Bucket=bucket, Key=key)
    metadata = obj['Metadata']
    print(metadata)
    try:
        assert all(
            key in metadata for key in (
                'x-amz-key-v2', 'x-amz-iv', 'x-amz-iv', 'x-amz-matdesc', 'x-amz-unencrypted-content-length'
            )
        )
    except AssertionError as e:
        capture_exception(e)
        raise EmailDecryptionException({'detail': str(e)})

    encrypted_envelope_key = base64.b64decode(metadata['x-amz-key-v2'])

    iv = base64.b64decode(metadata['x-amz-iv'])

    encryption_context = json.loads(metadata['x-amz-matdesc'])
    original_size = int(metadata['x-amz-unencrypted-content-length'])
    print("All kms information available")

    # Decrypt the envelope key with the CMK
    encryption_key = kms.decrypt(
        CiphertextBlob=encrypted_envelope_key,
        EncryptionContext=encryption_context
    )['Plaintext']

    # Maximum size of email is 30MB so we can read this into memory
    encrypted_text = obj['Body'].read()

    aesgcm = AESGCM(encryption_key)

    print(aesgcm)

    plain_text_bytes = aesgcm.decrypt(iv, encrypted_text, None)

    try:
        assert len(plain_text_bytes) == original_size
    except AssertionError as e:
        capture_exception(e)
        raise EmailDecryptionException({'detail': str(e)})

    return plain_text_bytes.decode('utf-8')


# Create session using your current creds
boto_sts=boto3.client('sts')

# Request to assume the role like this, the ARN is the Role's ARN from
# the other account you wish to assume. Not your current ARN.
stsresponse = boto_sts.assume_role(
    RoleArn="<role-arn>",
    RoleSessionName='<session-name>'
)

# Save the details from assumed role into vars
newsession_id = stsresponse["Credentials"]["AccessKeyId"]
newsession_key = stsresponse["Credentials"]["SecretAccessKey"]
newsession_token = stsresponse["Credentials"]["SessionToken"]

bucket='<bucket-name>'
key='<bucket-key-path>'
get_content=get_data_from_s3(bucket, key)
#print or write to file
print(get_content)

```
