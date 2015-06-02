# hello-client-eureka
Makes a call to hello-service with a URI provided by the **Service Registry for Pivotal Cloud Foundry**.

## hello-service setup
This app requires hello-service-eureka to be running. In this example we'll do a blue-green deployment of hello-service-eureka, and consume that in hello-client-eureka.

1. Shut down any running instances of hello-service.
1. Run the following commands to start both a blue and a green deployment of hello-service-eureka:

  ```
cd ../hello-service-eureka
mvn package
cf push hello-service-blue --no-start
cf set-env hello-service-blue SPRING_PROFILES_ACTIVE blue
cf start hello-service-blue
cf push hello-service-green --no-start
cf set-env hello-service-green SPRING_PROFILES_ACTIVE green
cf start hello-service-green
  ```

## To run on your CF environment:
```cd``` to the ```hello-client-eureka``` folder and ```mvn clean package && cf push``` to push the app as hello-client to the currently targeted space, using a random route

## To test the app:
``` curl [the random route outputted by cf push] ```

You should see the value returned by either ```hello-service-blue``` ("Hello blue!") or ```hello-service-green``` ("Hello green!"), and this should switch each time you run ```curl```. The switching occurs due to the Ribbon internal load balancer that the [RestTemplate uses under the hood](https://github.com/willtran-/spring-cloud-demo/blob/b2cbefeb8bdcf7edf6b21b36aff9c96c4365cedd/hello-client-eureka/src/main/java/demo/HelloClientEureka.java#L36-L41). This RestTemplate is autoconfigured for your application when you use ```@EnableServiceDiscovery```.

If you stop either  ```hello-service-blue``` or ```hello-service-green```, for a minute you'll see responses for both the live server and errors due to the other server not being reachable. These errors can be avoided in the [hello-client-hystrix](https://github.com/willtran-/spring-cloud-demo/hello-client-hystrix) example. Try starting and stopping each of the blue and green services. 