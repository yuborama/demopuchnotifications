<script lang="ts" setup>
import { defineComponent, onMounted, ref, watch } from "vue";
import { Geolocation } from "@capacitor/geolocation";
import { InAppBrowser } from "@capgo/inappbrowser";
import { Device } from "@capacitor/device";
import {
  ActionPerformed,
  PushNotificationSchema,
  PushNotifications,
  Token,
} from "@capacitor/push-notifications";
const loc = ref<{
  lat: null | number;
  long: null | number;
}>({
  lat: null,
  long: null,
});
const tokennotification = ref<string | null>(null);
const getCurrentPosition = async () => {
  const pos = await Geolocation.getCurrentPosition();
  loc.value = {
    lat: pos.coords.latitude,
    long: pos.coords.longitude,
  };
};
const openWebView = () => {
  console.log("openWebView");
  InAppBrowser.open({
    url: "https://www.google.com",
  });

  InAppBrowser.addListener("closeEvent", (event) => {
    alert("loadstart: " + JSON.stringify(event));
  });
  InAppBrowser.addListener("urlChangeEvent", (event) => {
    alert("urlChangeEvent: " + JSON.stringify(event));
  });
};

const logDeviceInfo = async () => {
  const info = await Device.getInfo();

  if (info.platform === "ios") {
    return `${info.platform}-${info.osVersion}`;
  }
  if (info.platform === "android") {
    return `${info.platform}-${info.osVersion}`;
  }
  const userAgent = window.navigator.userAgent;
  return `${info.platform}-${userAgent}`;
};

onMounted(() => {
  PushNotifications.requestPermissions().then((result) => {
    if (result.receive === "granted") {
      // Register with Apple / Google to receive push via APNS/FCM
      PushNotifications.register();
    } else {
      // Show some error
    }
  });
  // On success, we should be able to receive notifications
  PushNotifications.addListener("registration", (token: Token) => {
    tokennotification.value = token.value;
    alert("Push registration success, token: " + token.value);
  });
  // Some issue with our setup and push will not work
  PushNotifications.addListener("registrationError", (error: any) => {
    alert("Error on registration: " + JSON.stringify(error));
  });
  // Show us the notification payload if the app is open on our device
  PushNotifications.addListener(
    "pushNotificationReceived",
    (notification: PushNotificationSchema) => {
      alert("Push received: " + JSON.stringify(notification));
    }
  );
  // Method called when tapping on a notification
  PushNotifications.addListener(
    "pushNotificationActionPerformed",
    (notification: ActionPerformed) => {
      alert("Push action performed: " + JSON.stringify(notification));
    }
  );
});
</script>

<template>
  <div>
    <h1>Geolocation</h1>
    <p>Your location is:</p>
    <p>Latitude: {{ loc.lat }}</p>
    <p>Longitude: {{ loc.long }}</p>
    <p>Token: {{ tokennotification }}</p>
    <input type="text" v-model="tokennotification" />
    <button @click="getCurrentPosition">Get Current Location</button>
    <button @click="openWebView">Open WebView</button>
    <button @click="logDeviceInfo">get info</button>
  </div>
</template>
