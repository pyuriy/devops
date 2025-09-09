### Mastering the RHEL 7 to 8 Upgrade with Leapp: A Step-by-Step Guide

*August 16, 2024 | 3:37 PM EDT*

Upgrading from Red Hat Enterprise Linux (RHEL) 7 to 8 can be a game-changer for your infrastructure, bringing enhanced security, performance, and modern features. Using the Leapp tool, this in-place upgrade is more manageable than ever—but it requires careful planning to avoid pitfalls. Based on my hands-on experience, here’s a concise, practical guide to navigate the process smoothly, along with tips to troubleshoot common issues.

#### 1. Prepare Your System
Start by updating your RHEL 7 OS to the latest version and rebooting:
- Run `yum -y update` and `systemctl reboot`.
- Check logs for errors in `/var/log/leapp/leapp-report.txt` if issues arise.

**Tip**: Install the Leapp module from Ansible Automation Hub (`rpm -Uvh https://console.redhat.com/ansible/automation-hub/repo/validated/infra/leapp/`) for added reliability.

Disable all repositories except `redhat.repo` to prevent conflicts:
```bash
# for f in `ls epel*`; do mv $f $f.bk; done
```
Verify with `ll` to ensure only `redhat.repo` remains active.

#### 2. Secure Your Backups
Before proceeding, create a backup of critical data:
- Archive `/etc`: `tar czvf etc_$(hostname)_$(date +"%Y%m%d%H%M").tar.gz /etc`.
- Generate a system report: `sosreport -q --tmp-dir /tmp` (install with `yum install sos` if needed).
- Copy backups to your laptop using `rsync` or `scp` for safekeeping.

**Pro Tip**: Compare pre- and post-upgrade configs with a script like `compare_etc_after_os_patching.pl` to spot changes.

#### 3.  SELinux
- Set SELinux to permissive: Edit `/etc/selinux/config` to `SELINUX=permissive` and confirm with `getenforce`.
- set it to disable is the best to make the upgrdae proccess quicker

#### 4. Register the Correct Subscription
Clean and re-register your subscription to align with RHEL 8:
```bash
subscription-manager remove --all
subscription-manager unregister
subscription-manager clean
yum clean all
subscription-manager register --org="Your_Org" --activationkey="leapp_rhel7_8"
subscription-manager refresh
```
Ensure only `redhat.repo` is active post-registration.

#### 5. Disable Cloud-Init and Reboot
Prevent interference from cloud-init:
- Disable it: `systemctl disable cloud-init` and `touch /etc/cloud/cloud-init.disabled`.
- Reboot to confirm no impact: `systemctl reboot`.

#### Troubleshooting Common Challenges
Even with preparation, upgrades can hit snags. Here’s how to tackle them:
- **Boot Space Issues**: Clear old kernels with `package-cleanup --oldkernels --count=1` (RHEL 7) or `dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)` (RHEL 8). Regenerate GRUB with `grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg`.
- **Leapp Errors**: Check `/var/log/leapp/answerfile` for inhibitors (e.g., pam_pkcs11) and resolve with `leapp answer --section remove_pam_pkcs11_module_check.confirm=True`.
- **Network Renaming**: Set `export LEAPP_NO_NETWORK_RENAMING=1` to preserve NIC names.
- **Post-Upgrade Logs**: Archive logs with `tar czvf /tmp/$(hostname)_$(date +"%Y-%m-%d_%H-%M")-leap-upgrade-logs.tgz /var/log/leapp/` for auditing.

#### Post-Upgrade Steps
After rebooting, update to the latest RHEL 8:
- Re-register: `subscription-manager register --org="IaaS_Org" --activationkey="iaas-rhel8"`.
- Update: `yum -y update`.
- Apply security fixes (e.g., SSH hardening, etc)

This process has streamlined upgrades in my production environments, turning potential headaches into successes. Test in a staging setup first, and consider automating with Ansible for consistency.

Have you faced unique challenges during a RHEL upgrade? Share your experiences or questions below.
I’d love to connect and help! Like, share, or follow for more IT insights.

#RHEL #LinuxAdmin #SysAdmin #DevOps #Leapp #UpgradeGuide #ITOperations #RedHat
