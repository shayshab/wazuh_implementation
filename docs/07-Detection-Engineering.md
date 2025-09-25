## 07 — Detection Engineering

### Custom Decoders
```xml
<decoder name="myapp-decoder">
  <prematch>myapp:</prematch>
  <regex offset="after_prematch">client=(\S+) action=(\S+) status=(\d+)</regex>
  <order>client,action,status</order>
</decoder>
```

### Custom Rules
```xml
<rule id="100201" level="8">
  <if_decoder_name>myapp-decoder</if_decoder_name>
  <field name="action">login</field>
  <field name="status">\b(401|403)\b</field>
  <description>MyApp failed login</description>
  <mitre>
    <id>T1110</id>
  </mitre>
  <group>authentication, myapp,</group>
</rule>
```

### Testing & QA
- Feed sample logs; verify fields and rule hits
- Add unit-style tests where possible; version control decoders/rules

### MITRE Mapping & Catalog
- Map rules to techniques; maintain a coverage matrix
- Track false positives and tuning history
