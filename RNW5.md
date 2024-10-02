# :computer: Geolocation

## Initialize expo project

```bash
npx create-expo-app Geolocation --template blank
cd Geolocation
npx expo install expo-location react-native-maps
```

## App.js

```jsx
import React, { useEffect, useState } from 'react';
import { StyleSheet, Text, View, Dimensions } from 'react-native';
import * as Location from 'expo-location';
import MapView, { Marker } from 'react-native-maps';

export default function App() {
  const [location, setLocation] = useState(null);
  const [errorMsg, setErrorMsg] = useState(null);

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== Location.PermissionStatus.GRANTED) {
        setErrorMsg('Permission to access location was denied');
        return;
      }

      let location = await Location.getCurrentPositionAsync({});
      setLocation(location.coords);
    })();
  }, []);

  let text = 'Waiting for location...';
  if (errorMsg) {
    text = errorMsg;
  } else if (location) {
    text = `Latitude: ${location.latitude}, Longitude: ${location.longitude}`;
  }

  return (
    <View style={styles.container}>
      {location ? (
        <MapView
          style={styles.map}
          initialRegion={{
            latitude: location.latitude,
            longitude: location.longitude,
            latitudeDelta: 0.005,
            longitudeDelta: 0.005,
          }}
        >
          <Marker
            coordinate={{ latitude: location.latitude, longitude: location.longitude }}
            title={"Your Location"}
          />
        </MapView>
      ) : (
        <Text>{text}</Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  map: {
    width: Dimensions.get('window').width,
    height: Dimensions.get('window').height,
  },
});
```

## Update app.json

Add the permissions for iOS and Android by adding the following in the ```app.json``` file:

```
  "ios": {
    "infoPlist": {
      "NSLocationAlwaysUsageDescription": "We need to display your location on the home screen.",
      "NSLocationWhenInUseUsageDescription": "This app uses your location to show your position on the map."
    }
  },
  "android": {
    "permissions": ["ACCESS_COARSE_LOCATION", "ACCESS_FINE_LOCATION"]
  }
  
 ```

## Run 

```npx expo start```

- Use a physical device to check if the map shows your current location.
- ```console.log``` your current position and verify (lat, long) using this reverse geocoding site: https://www.latlong.net/Show-Latitude-Longitude.html

Optional:
- Use reverse geocoding api, e.g. https://apidocs.geoapify.com/docs/geocoding/reverse-geocoding/#quick-start to show your address in the console.
- You will have to obtain an API key to send a request to geoapify reverse geocoding. Example request: ``` `https://api.geoapify.com/v1/geocode/reverse?lat=${coords.latitude}&lon=${coords.longitude}&format=json&apiKey=${API_KEY}` ```


# :computer: Geofence

## Initialize expo project

```bash
npx create-expo-app Geofence --template blank
cd Geofence
npx expo install expo-location react-native-maps
```

## App.js

```jsx
import React, { useState, useEffect } from "react";
import { StyleSheet, Text, View, Dimensions } from "react-native";
import MapView, { Marker, Circle } from "react-native-maps";
import * as Location from "expo-location";

export default function App() {
  const [location, setLocation] = useState(null);
  const [errorMsg, setErrorMsg] = useState(null);

  // Geofence coordinates and radius (you can modify this to your preference)
  const geofenceLocation = {
    latitude: 45.2854,
    longitude: -75.91976,
  };
  const geofenceRadius = 1000; // Geofence radius in meters

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") {
        setErrorMsg("Permission to access location was denied");
        return;
      }

      let location = await Location.getCurrentPositionAsync({});
      setLocation(location);

      // Set up geofencing
      Location.startGeofencingAsync("GEOFENCE_TASK", [
        {
          latitude: geofenceLocation.latitude,
          longitude: geofenceLocation.longitude,
          radius: geofenceRadius,
        },
      ]);

      Location.watchPositionAsync(
        { accuracy: Location.Accuracy.Highest, distanceInterval: 100 },
        (newLocation) => {
          setLocation(newLocation);
          handleGeofencing(newLocation.coords);
        }
      );
    })();

    return () => {
      Location.stopGeofencingAsync("GEOFENCE_TASK");
    };
  }, []);

  const handleGeofencing = (coords) => {
    const distance = getDistanceFromLatLonInMeters(
      coords.latitude,
      coords.longitude,
      geofenceLocation.latitude,
      geofenceLocation.longitude
    );
    if (distance <= geofenceRadius) {
      console.log("Geofence Alert", "You are inside the geofence!");
    } else {
      console.log("Geofence Alert", "You are outside the geofence!");
    }
  };

  // Utility function to calculate the distance between two coordinates
  const getDistanceFromLatLonInMeters = (lat1, lon1, lat2, lon2) => {
    const R = 6371e3; // Radius of Earth in meters
    const dLat = ((lat2 - lat1) * Math.PI) / 180;
    const dLon = ((lon2 - lon1) * Math.PI) / 180;
    const a =
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos((lat1 * Math.PI) / 180) *
        Math.cos((lat2 * Math.PI) / 180) *
        Math.sin(dLon / 2) *
        Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c; // Distance in meters
    return distance;
  };

  if (errorMsg) {
    return <Text>{errorMsg}</Text>;
  }

  if (!location) {
    return <Text>Fetching location...</Text>;
  }

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: geofenceLocation.latitude,
          longitude: geofenceLocation.longitude,
          latitudeDelta: 0.05,
          longitudeDelta: 0.05,
        }}
      >
        <Marker
          coordinate={{
            latitude: geofenceLocation.latitude,
            longitude: geofenceLocation.longitude,
          }}
          title="Geofence Center"
          description="This is the center of the geofence"
        />
        <Circle
          center={geofenceLocation}
          radius={geofenceRadius}
          strokeWidth={2}
          strokeColor="rgba(0, 150, 255, 0.5)"
          fillColor="rgba(0, 150, 255, 0.3)"
        />
        {location && (
          <Marker
            coordinate={{
              latitude: location.coords.latitude,
              longitude: location.coords.longitude,
            }}
            title="Your Location"
            pinColor="green"
          />
        )}
      </MapView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  map: {
    width: Dimensions.get("window").width,
    height: Dimensions.get("window").height,
  },
});

```

## Update app.json

Add the permissions for iOS and Android by adding the following in the ```app.json``` file:

```
  "ios": {
    "infoPlist": {
      "NSLocationAlwaysUsageDescription": "We need to display your location on the home screen.",
      "NSLocationWhenInUseUsageDescription": "This app uses your location to show your position on the map."
    }
  },
  "android": {
    "permissions": ["ACCESS_COARSE_LOCATION", "ACCESS_FINE_LOCATION"]
  }
  
 ```

## Run 

```npx expo start```

- Use a physical device to check if the map shows your current location.
- Set the center of the geofence to your current location. 
- See if you current location triggers the geofence

# :computer: ImagePicker

## Initialize expo project

```bash
npx create-expo-app ImagePicker --template blank
cd ImagePicker
npx expo install expo-image-picker
```

## App.js

```jsx
import React, { useState } from "react";
import { Image, Pressable, StyleSheet, View, Text } from "react-native";
import * as ImagePicker from "expo-image-picker";
// import * as Permissions from "expo-permissions";

export default function App() {
  const [image, setImage] = useState(null);

  const pickImage = async () => {
    // Ask for permission to access the gallery
    const permissionResult =
      await ImagePicker.requestMediaLibraryPermissionsAsync();

    if (permissionResult.granted === false) {
      alert("Permission to access the gallery is required!");
      return;
    }

    // Open the image picker
    let result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsEditing: true,
      aspect: [4, 3],
      quality: 1,
    });

    if (!result.canceled) {
      setImage(result.assets[0].uri);
    }
  };

  const takePhoto = async () => {
    const permissionResult = await ImagePicker.requestCameraPermissionsAsync();

    if (permissionResult.granted === false) {
      alert("Permission to access the camera is required!");
      return;
    }

    let result = await ImagePicker.launchCameraAsync({
      allowsEditing: true,
      aspect: [4, 3],
      quality: 1,
    });

    if (!result.canceled) {
      setImage(result.assets[0].uri);
    }
  };

  return (
    <View style={styles.container}>
      <Pressable onPress={pickImage} style={styles.button}>
        <Text style={styles.text}>Gallery</Text>
      </Pressable>
      <Pressable onPress={takePhoto} style={styles.button}>
        <Text style={styles.text}>Camera</Text>
      </Pressable>
      {image && <Image source={{ uri: image }} style={styles.image} />}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  image: {
    width: 300,
    height: 300,
    marginTop: 20,
  },
  button: {
    paddingVertical: 12, // Set vertical padding
    paddingHorizontal: 20, // Set horizontal padding
    marginBottom: 20,
    backgroundColor: "#2f9ad0",
    borderRadius: 10,
  },
  text: {
    color: "white",
    fontSize: 16,
  },
});
```

## Update app.json

Your app.json should have the following permissions:

```json
"ios": {
  "supportsTablet": true,
  "infoPlist": {
    "NSMicrophoneUsageDescription": "Want to access your microphone because...",
    "NSPhotoLibraryUsageDescription": "Want to access your photo library because...",
    "NSCameraUsageDescription": "Want to use your camera because..."
  }
},
"android": {
  "adaptiveIcon": {
    "foregroundImage": "./assets/adaptive-icon.png",
    "backgroundColor": "#FFFFFF"
  },
  "permissions": [
    "CAMERA",
    "READ_EXTERNAL_STORAGE",
    "WRITE_EXTERNAL_STORAGE"
  ]
},
"plugins": [
  [
    "expo-image-picker",
    {
      "photosPermission": "The app accesses your photos to let you share them with your friends.",
      "cameraPermission": "Please iOS let me access the camera."
    }
  ]
],
 ```

## Run 

```npx expo start```

# :computer: SwipeToDelete

## Initialize expo project

```bash
npx create-expo-app SwipeToDelete --template blank
cd SwipeToDelete
npm install react-native-gesture-handler
```

## App.js

```jsx
import React, { useState } from "react";
import { SafeAreaView, FlatList, Text, View, StyleSheet } from "react-native";
import {
  GestureHandlerRootView,
  Swipeable,
} from "react-native-gesture-handler";
import { TouchableOpacity } from "react-native-gesture-handler";

export default function App() {
  const [data, setData] = useState([
    { id: "1", text: "Item 1" },
    { id: "2", text: "Item 2" },
    { id: "3", text: "Item 3" },
    { id: "4", text: "Item 4" },
  ]);

  const deleteItem = (id) => {
    setData(data.filter((item) => item.id !== id));
  };

  const renderRightActions = (id) => (
    <TouchableOpacity
      onPress={() => deleteItem(id)}
      style={styles.deleteButton}
    >
      <Text style={styles.deleteText}>Delete</Text>
    </TouchableOpacity>
  );

  const renderItem = ({ item }) => (
    <Swipeable renderRightActions={() => renderRightActions(item.id)}>
      <View style={styles.listItem}>
        <Text style={styles.itemText}>{item.text}</Text>
      </View>
    </Swipeable>
  );

  return (
    <GestureHandlerRootView style={styles.container}>
      <SafeAreaView>
        <FlatList
          data={data}
          keyExtractor={(item) => item.id}
          renderItem={renderItem}
        />
      </SafeAreaView>
    </GestureHandlerRootView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    paddingTop: 50,
  },
  listItem: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ccc",
    backgroundColor: "#fff",
  },
  itemText: {
    fontSize: 18,
  },
  deleteButton: {
    backgroundColor: "red",
    justifyContent: "center",
    alignItems: "center",
    width: 80,
    height: "100%",
  },
  deleteText: {
    color: "#fff",
    fontWeight: "bold",
  },
});
```

## Run 
```npx expo start```

# :computer: AsyncStorage

## Initialize expo project
```bash
npx create-expo-app AsyncStorage --template blank
cd AsyncStorage
npx expo install @react-native-async-storage/async-storage
```

## App.js

```jsx
import React, { useState, useEffect } from "react";
import { View, Text, TextInput, Button, StyleSheet, Alert } from "react-native";
import AsyncStorage from "@react-native-async-storage/async-storage";

export default function App() {
  const [inputValue, setInputValue] = useState("");
  const [storedValue, setStoredValue] = useState("");

  // Load stored value on component mount
  useEffect(() => {
    const loadStoredValue = async () => {
      try {
        const value = await AsyncStorage.getItem("myKey");
        if (value !== null) {
          setStoredValue(value);
        }
      } catch (e) {
        console.error("Failed to load data", e);
      }
    };
    loadStoredValue();
  }, []);

  // Store value in AsyncStorage
  const storeData = async () => {
    try {
      await AsyncStorage.setItem("myKey", inputValue);
      setStoredValue(inputValue);
      Alert.alert("Success", "Data saved successfully!");
    } catch (e) {
      console.error("Failed to save data", e);
    }
  };

  // Clear stored value from AsyncStorage
  const clearData = async () => {
    try {
      await AsyncStorage.removeItem("myKey");
      setStoredValue("");
      Alert.alert("Success", "Data cleared successfully!");
    } catch (e) {
      console.error("Failed to clear data", e);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>AsyncStorage Demo</Text>

      <TextInput
        style={styles.input}
        placeholder="Type something..."
        value={inputValue}
        onChangeText={setInputValue}
      />

      <Button title="Save" onPress={storeData} />

      <View style={styles.spacer} />

      {storedValue ? (
        <View>
          <Text style={styles.text}>Stored Value: {storedValue}</Text>
          <Button title="Clear" onPress={clearData} />
        </View>
      ) : (
        <Text style={styles.text}>No data stored yet</Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  input: {
    height: 40,
    borderColor: "gray",
    borderWidth: 1,
    width: "100%",
    marginBottom: 20,
    padding: 10,
  },
  text: {
    fontSize: 18,
    marginVertical: 10,
  },
  spacer: {
    height: 20,
  },
});
```

## Run 

```npx expo start```

- Store some value into asycstorage.
- Kill the app
- Open the app again and confirm that the value was stored
