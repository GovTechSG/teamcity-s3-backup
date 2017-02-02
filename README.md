# Backup Teamcity Settings

## Pre-requisites
### Software on Ansible Machine
You will need the following installed on the machine:
 - [Ansible 2.2](https://docs.ansible.com/ansible/intro_installation.html)
 - [AWS CLI](https://github.com/aws/aws-cli)

### Amazon S3
You will need to setup a S3 bucket with the appropriate settings.

Then, you should create a IAM user specifically with permissions to only put objects into the bucket. An example Policy
document is this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::YOUR-BUCKET-NAME"
            ]
        },
        {
            "Sid": "2",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload"
            ],
            "Resource": [
                "arn:aws:s3:::YOUR-BUCKET-NAME/*"
            ]
        }
    ]
}
```

### TeamCity
You should also setup a TeamCity user solely for the purpose of trigerring and downloading backups only. The user will
need the `Change backup settings and control backup process` permission.

### Ansible Secrets
Refer to `secrets.template.yml` and fill it in. By default, the playbook will look for the secrets in `secrets.yml`, but
you can override it with the `secrets_file` variable.

### Playbook Variables
The playbook has additional variables you can override at run time, in addition to the secrets file it will include. Refer
to the playbook.

## Running the entire playbook

You won't need to ask ansible to run the playbook at another host, and you can run it locally. The default invocation
would simply be

```bash
ansible-playbook -i localhost --connection local site.yml
```

### Tags
The playbook tasks occur in two separate and distinct stages that can be run independently. Each stage is tagged.

 - Trigger Backup on TeamCity (`trigger_backup`)
 - Download Backup from TeamCity and upload to S3 (`upload_backup`)

For instance, to only trigger backup, do

```bash
ansible-playbook -i localhost --connection local site.yml --skip-tags upload_backup
```

If you just want to upload a backup to S3, you will need to know the name of the backup and then do:
```bash
ansible-playbook -i localhost --connection local site.yml \
  -e "{'teamcity': { 'backup_name': 'TeamCity_Backup_2017-02-02T05-28-36Z_20170202_052905.zip'}}" \
  --skip-tags trigger_backup
```
