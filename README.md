# Linux Australia Gitlab Ansible Playbook

## Steps to complete after initial playbook run

1. Disable new account registrations (if you aren't yet ready for public registrations)
2. Change the root user's email address to something you control/want it to be
3. Configure networking to use the proxy host as the network gateway (to ensure source IPs are maintained rather than NAT'd)

## Steps to restore from a backup

1. Install ansible `apt update && apt install ansible`
2. Pull down this repo and `cd` into it
3. Run Ansible Playbook as root: `# ansible-playbook -i ./hosts ./gitlab.yml -b --ask-vault-pass -l gitlab` and provide the vault passphrase
4. Extract the `etc-gitlab` tar backup and move the `gitlab-secrets.json` back to `/etc/gitlab/`
5. Run the Gitlab reconfiguration command to ensure it has loaded the secrets: `gitlab-ctl reconfigure`
6. Copy the Gitlab data backup (named something like `EPOCH_YYYY_MM_DD_GitLab_version_gitlab_backup.tar`) into `/srv/backups`
7. Stop the processes that connect to the database, but leave Gitlab running: `sudo gitlab-ctl stop unicorn && sudo gitlab-ctl stop sidekiq` and verify the status: `sudo gitlab-ctl status`
8. Run the restore command, substituting the value for `BACKUP` with the prefix from the backup you wish to restore: `gitlab-rake gitlab:backup:restore BACKUP=1534653088_2018_08_19_11.1.4-ee`
9. Agree to dropping the table (type `yes` when prompted), agree to rebuilding the authorized_keys file (`yes` when prompted).
10. Restart Gitlab `gitlab-ctl restart`
11. Check Gitlab is OK `gitlab-rake gitlab:check SANITIZE=true`
12. Check login works OK with MFA/2FA, that repo data and issues are restored, and the push/pull via SSH works.
13. Finished.

For more information/steps about running a restore see: https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-for-omnibus-installations
