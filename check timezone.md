# 调用 timedatectl 查看机器时区，如果是 Asia/Singapore 输出 timezone_check_passed，否则输出 timezone_check_failed。

---

```yaml
- hosts: test-server1
  tasks:
    - name: Test message
      debug:
        msg: Hello Ansible!
    - name: Get current timezone
      command: timedatectl show -p Timezone --value
      register: timezone

    - name: Fail if timezone is not Asia/Singapore
      assert:
        that: timezone.stdout.strip() == "Asia/Singapore"
        fail_msg: "timezone_check_failed"
        success_msg: "timezone_check_passed"

# 执行结果
ansible-playbook test.yml

PLAY [test-server1] ******************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************
ok: [test-server1]

TASK [Test message] ******************************************************************************************************************
ok: [test-server1] => 
  msg: Hello Ansible!

TASK [Get current timezone] **********************************************************************************************************
changed: [test-server1]

TASK [Fail if timezone is not Asia/Singapore] ****************************************************************************************
ok: [test-server1] => changed=false 
  msg: timezone_check_passed

PLAY RECAP ***************************************************************************************************************************
test-server1               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0