# This is the schema definition for a valid PX configuration file.  This file is expected to be available at `/etc/pwx/config.json`

```
{
 "description": "PX config json schema",
 "type": "object",
 "properties": {
   "version": {
     "type": "string"
   },
   "created": {
     "type": "string"
   },
   "clusterid": {
     "type": "string"
   },
   "mgtiface": {
     "type": "string"
   },
   "dataiface": {
     "type": "string"
   },
   "kvdb": {
     "type": "array",
     "minItems": 1,
     "items": {"type": "string", "format": "uri" },
     "uniqueItems": true
   },
   "loggingurl": {
     "type": "string"
   },
   "alertingurl": {
     "type": "string"
   },
   "storage": {
     "type": "object",
     "properties": {
       "devices_md": {
         "type": "array",
         "items": { "type": "string" },
         "uniqueItems": true
       },
       "devices": {
         "type": "array",
         "minItems": 1,
         "items": { "type": "string" },
         "uniqueItems": true
       }
     }
   },
   "raidlevel": {
     "type": "string"
   },
   "raidlevelmd": {
     "type": "string"
   },
   "driver": {
     "type": "string"
   },
   "debug_level": {
     "type": "string"
   },
   "domain": {
     "type": "string"
   }
 }
}
```
