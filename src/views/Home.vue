<template>
  <div class="home">
    <h1>Scan device</h1>
    <button @click="scanAndStop">scan</button>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      battery: 0,
      temperature: 0,
      humidity : 0,
      batteryTension : 0,
      handleNotifDone : false
    };
  },
  name: "Home",
  methods: {
    async handleNotifications(event) {
      let value = event.target.value;
      let a = [];
      for (let i = 0; i < value.byteLength; i++) {
        a.push(("00" + value.getUint8(i).toString(16)).slice(-2));
      }
      console.log(a.join(" "));
      let temperatureHex = "";
      let humidityHex = a[2]+"";
      let batteryHex = "";
      if(a[0]<a[1]){
        temperatureHex = a[0]+""+a[1];
      }
      else{
        temperatureHex = a[1]+""+a[0];
      }
      if(a[3]<a[4]){
        batteryHex = a[3]+""+a[4];
      }
      else{
        batteryHex = a[4]+""+a[3];
      }
      let temperature = parseInt(temperatureHex,16)/100;
      let humidity = parseInt(humidityHex,16);
      let batteryTension = parseInt(batteryHex,16)/1000;
      this.temperature = temperature;
      this.humidity = humidity;
      this.batteryTension = batteryTension;
      
    },
    async scanAndStop() {
      try {
        let filters = [];
        const filterNamePrefix = "LYWS";
        filters.push({ namePrefix: filterNamePrefix });
        let options = {};
        options.filters = filters;
        options.optionalServices = [
          "battery_service",
          "device_information",
          "ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6",
        ];
        var device = await navigator.bluetooth.requestDevice(options);
        console.log("Connecting to GATT server");
        let server = null;
        try {
          server = await device.gatt.connect();
        } catch (error) {
          console.log("[gatt error]", error.messsage);
          server = await device.gatt.connect();
        }
        /// battery service
        const batteryService = await server.getPrimaryService(
          "battery_service"
        );
        const batteryCharacteristic = await batteryService.getCharacteristic(
          "battery_level"
        );
        const batteryValue = await batteryCharacteristic.readValue();
        console.log("> Battery Level is " + batteryValue.getUint8(0) + "%");
        this.battery = batteryValue.getUint8(0);

        /// device information service
        const informationService = await server.getPrimaryService(
          "device_information"
        );
        const modalCharacteristic = await informationService.getCharacteristic(
          "model_number_string"
        );
        const modalValue = await modalCharacteristic.readValue();
        console.log("> Model Number String " + modalValue.getUint8(0));

        /// beacon information service
        const beaconService = await server.getPrimaryService(
          "ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6"
        );
        const beaconCharacteristic = await beaconService.getCharacteristic(
          "ebe0ccc1-7a0a-4b0c-8a1a-6ff2997da3a6"
        );
        // const beaconValue = await beaconCharacteristic.readValue();
        await beaconCharacteristic.startNotifications();
        console.log("> Notifications started");
        beaconCharacteristic.addEventListener(
          "characteristicvaluechanged",
          this.handleNotifications
        );
        this.handleNotifDone = true;
      } catch (error) {
        // console.log(service);
        // }
        console.log("[home] scan error " + error);
      }
    },
  },
};
</script>
