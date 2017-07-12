---
layout: post
title:  "Using Spring cloud consul."
date:   2017-07-11 20:52:29 -0700
categories: jekyll update
---
# Registering spring boot applications with spring boot using spring cloud consul.

Service discovery can be achieved via Consul.(https://www.consul.io/)

Spring boot application can register with Consul using spring-cloud-consul.(https://cloud.spring.io/spring-cloud-consul/)

You can set up consul by installing it or if you use docker you can set it up via the command
``` docker run -p 8500:8500 consul:0.8.5 ```

If you set it up via docker you can go to
http://localhost:8500/ui/#/dc1/services and you should see


![consul](/assets/consul-web.png)

Now we will set up two spring boot webservices
* Pet Service
* Pet Client

![consul-registration](/assets/Consul.jpg)

They both register with consul when we include the dependency

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

Consul needs some health checks to pass for these services to register successfully, so we include spring-boot-actuator to provide these health checks
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```


After this you should see the services registered in Consul.

![consul-after-reg](/assets/consul-after-reg.png)

AddingMake sure you add the annotation
```
@EnableDiscoveryClient
```

Once this is all set up the client can talk to the service via service discovery.

```
  @GetMapping("/{name}")
    public String[] getPets(@PathParam("name") String petName)
    {
        ServiceInstance serviceInstance=client.getInstances("pet-service")
                .stream().findFirst().
                        orElseThrow(()-> new RuntimeException("pet-service not found"));

        String url = serviceInstance.getUri().toString()+ "/pets/"+petName;
        return restTemplate.getForObject(url,String[].class);
    }
```

And walla, you are all done.
All code can be found at
https://github.com/arnabmitra/Consul-demo