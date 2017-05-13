# Linux routing 

Example of a private network routed through redundant Linux boxes. 

`outside` mimics the network with static routes balanced over `router1` and `router2`. Inside will route through the VIP address. 

## Configuration

* `inside` default gateway is `172.16.0.1`
* `ouside` gateway to `172.16.0.0/24` is both `10.0.0.2` and `10.0.0.3`
* `router1` and `router2` configured with HA 
  * HA setup with UCARP 
  * VIP address `172.16.0.1`

## Sketch 
                                                                                 
```                                                                              
                                    +---------+                                  
                         172.16.0.2 |         | 10.0.0.2                         
                       +------------+ router1 +----------+                       
                       | 172.16.0.1 |         |          |                       
+--------+             |   (vip)    +---------+          |           +---------+ 
|        | 172.16.0.10 |                                 | 10.0.0.10 |         | 
| inside |-------------+                                 +-----------+ outside | 
|        |             |                                 |           |         | 
+--------+             |            +---------+          |           +---------+ 
                       |            |         |          |                       
                       +------------+ router2 +----------+                       
                         172.16.0.3 |         | 10.0.0.3                         
                                    +---------+                                  
```
