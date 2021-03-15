# SMA-Speedwire
SMA-Speedwire is an api for communicating with SMA products over Ethernet using the speedwire interface.  

It allows listening for exchanged data between your devices and logging meter readings.

Technical documents covering Speedwire can be found under:
* [Speedwire-TI-en-11.pdf](https://files.sma.de/downloads/Speedwire-TI-en-11.pdf) - 
  Speedwire technical information  
* [EMETER-Protokoll-TI-en-10.pdf](https://www.sma.de/fileadmin/content/global/Partner/Documents/SMA_Labs/EMETER-Protokoll-TI-en-10.pdf) - 
  SMA Energy Meter Protocol  
* [SpeedwireDD-TI-en-10.pdf](https://www.sma.de/fileadmin/content/global/Partner/Documents/sma_developer/SpeedwireDD-TI-en-10.pdf) - 
  Speedwire device discovery  

## Supported Products

* **SMA Energy Meter**  
  
* **Sunny Home Manager (2.0)**

* ...

I'm looking into implementing more products, but I need your help for this!  
Contribute a Telegram implementation for your device by opening a Pull Request or help in the development process by 
sharing packet captures.
Open an issue with your device to get started.

## Known issues

Speedwire uses UDP as transport protocol for sending packets to multicast groups.  
Therefore, you will need to make sure that routers between the device you want to communicate with, and your device have
multicast forwarding enabled.  
Some cheap switches are also known to cause problems with multicast (like blocking random groups or not allowing it at all), 
so make sure your hardware supports it.  

I would advise you to first test establishing a connection using the [DeviceDiscovery](samples/src/DeviceDiscovery.java) sample.

## Usage
Reading incoming data from an SMA Energy Meter / SMA Sunny Home Manager:  

```java
        Speedwire speedwire = new Speedwire();
        speedwire.onError(Exception::printStackTrace);
        speedwire.onTimeout(() -> System.err.println("speedwire timeout"));
        speedwire.onData(data -> {
            if (data instanceof EnergyMeterTelegram) {
                EnergyMeterTelegram em = (EnergyMeterTelegram) data;
        
                //device information
                int SUSyID = em.getSUSyID();
                long SerNo = em.getSerNo().longValueExact();
                String ip = em.getOrigin().getHostAddress();
                System.out.printf("Device %d %d on port %s%n", SUSyID, SerNo, ip);
        
                //current power draw (in W)
                Quantity<Power> w = em.getData(EnergyMeterChannels.TOTAL_P_IN).to(Units.WATT);
                System.out.printf("Ingress Power: %s%n", w);
        
                //energy meter total power reading (in kWh)
                Quantity<Energy> powerReading = em.getData(EnergyMeterChannels.TOTAL_P_IN_SUM)
                .to(MetricPrefix.KILO(Units.WATT).multiply(Units.HOUR).asType(Energy.class));
                System.out.printf("Total power reading: %s%n", powerReading);
            }
        });
        speedwire.start();
```

For more information read the well documented [javadoc](https://github.com/joblo2213/SMA-Speedwire/deployments/activity_log?environment=github-pages)
or have a look at the [samples](samples/src).

## Libraries
These open source libraries were used to create this api:

*  [unitsofmeasurement/indriya](https://github.com/unitsofmeasurement/indriya) licensed under the following
   [license](https://github.com/unitsofmeasurement/indriya/blob/master/LICENSE)
