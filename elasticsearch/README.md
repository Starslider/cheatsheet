### Node step down
```
PUT _cluster/settings
{
  "transient" :{
      "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
   }
}
```
