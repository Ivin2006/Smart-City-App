# Smart-City-App
A React Native &amp; Expo mobile application for citizens to report city-wide issues (potholes, garbage, street lights) featuring voice messaging, image capture, and dynamic theme switching.
# 🏙️ Smart City Community App

This is a mobile application demo built using React Native and Expo. It allows citizens to report infrastructure issues directly to city officials.

### 🎥 Demo Features
* **Reporting System:** Select categories like Potholes, Garbage, or Street Lights.
* **Multimedia:** Capture photos and record voice messages directly in the app.
* **Theme Support:** Fully functional Light and Dark modes.
* **Navigation:** Integrated bottom tab and stack navigation.

### ⚙️ Installation
1. Install Expo Go on your mobile device.
2. Clone this repo.
3. Run `npm install`.
4. Run `npx expo start` and scan the QR code.

import React, { useState, createContext, useContext } from 'react';
import { View, Text, TouchableOpacity, TextInput, StyleSheet, SafeAreaView, ScrollView, useColorScheme } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { Ionicons } from '@expo/vector-icons';
import { Audio } from 'expo-av';
import * as ImagePicker from 'expo-image-picker';

// --- 1. THEME CONTEXT ---
const ThemeContext = createContext();

const ThemeProvider = ({ children }) => {
  const systemTheme = useColorScheme();
  const [themeMode, setThemeMode] = useState('system'); // 'light', 'dark', 'system'

  const activeTheme = themeMode === 'system' ? systemTheme : themeMode;
  const isDark = activeTheme === 'dark';

  const colors = {
    background: isDark ? '#121212' : '#F5F7FA',
    card: isDark ? '#1E1E1E' : '#FFFFFF',
    text: isDark ? '#FFFFFF' : '#000000',
    primary: '#007AFF',
    error: '#FF3B30',
  };

  return (
    <ThemeContext.Provider value={{ themeMode, setThemeMode, colors, isDark }}>
      {children}
    </ThemeContext.Provider>
  );
};

// --- 2. SCREENS ---

// Home Screen (Matches your "Smart City" dashboard)
function HomeScreen({ navigation }) {
  const { colors } = useContext(ThemeContext);
  return (
    <SafeAreaView style={[styles.container, { backgroundColor: colors.background }]}>
      <View style={styles.header}>
        <Text style={[styles.title, { color: colors.text }]}>Smart City</Text>
        <TouchableOpacity><Text style={{ color: colors.primary }}>Sign In</Text></TouchableOpacity>
      </View>
      
      <TouchableOpacity style={styles.mainBtn}><Text style={styles.btnText}>My Reports</Text></TouchableOpacity>
      
      <TouchableOpacity 
        style={[styles.mainBtn, { backgroundColor: colors.primary }]} 
        onPress={() => navigation.navigate('ReportAnIssue')}
      >
        <Ionicons name="megaphone-outline" size={20} color="white" />
        <Text style={styles.btnText}> Report an Issue</Text>
      </TouchableOpacity>

      <View style={styles.errorBox}>
        <Text style={{ color: colors.error, fontWeight: 'bold' }}>Community Reports</Text>
        <Text style={{ color: colors.error }}>Error: Network request failed</Text>
        <Text style={{ color: 'gray', fontSize: 10 }}>Tried: http://10.0.2.2:8000</Text>
      </View>
    </SafeAreaView>
  );
}

// Report Form Screen (The one with Voice/Image)
function ReportAnIssue() {
  const { colors } = useContext(ThemeContext);
  const [recording, setRecording] = useState();
  const [image, setImage] = useState(null);

  const pickImage = async () => {
    let result = await ImagePicker.launchCameraAsync({ allowsEditing: true, quality: 1 });
    if (!result.canceled) setImage(result.assets[0].uri);
  };

  async function startRecording() {
    try {
      await Audio.requestPermissionsAsync();
      const { recording } = await Audio.Recording.createAsync(Audio.RecordingOptionsPresets.HIGH_QUALITY);
      setRecording(recording);
    } catch (err) { console.error(err); }
  }

  async function stopRecording() {
    setRecording(undefined);
    await recording.stopAndUnloadAsync();
  }

  return (
    <ScrollView style={[styles.container, { backgroundColor: colors.background }]}>
      <TouchableOpacity onPress={pickImage} style={[styles.uploadBox, { backgroundColor: colors.card }]}>
        <Ionicons name="camera-outline" size={40} color="gray" />
        <Text style={{ color: 'gray' }}>{image ? "Photo Attached" : "Tap to Capture Photo"}</Text>
      </TouchableOpacity>

      <View style={styles.section}>
        <Text style={{ color: colors.text, fontWeight: 'bold' }}>Voice Message (Optional)</Text>
        <TouchableOpacity 
          onPress={recording ? stopRecording : startRecording} 
          style={[styles.recordBtn, { backgroundColor: recording ? colors.error : colors.primary }]}
        >
          <Text style={styles.btnText}>{recording ? "Stop & Attach" : "Start Recording"}</Text>
        </TouchableOpacity>
      </View>

      <TextInput 
        placeholder="Please describe the issue in detail..." 
        placeholderTextColor="gray"
        multiline
        style={[styles.input, { backgroundColor: colors.card, color: colors.text }]} 
      />

      <TouchableOpacity style={[styles.mainBtn, { backgroundColor: colors.primary }]}>
        <Text style={styles.btnText}>Submit Report</Text>
      </TouchableOpacity>
    </ScrollView>
  );
}

// Settings Screen (The Theme Switcher)
function SettingsScreen() {
  const { colors, setThemeMode, themeMode } = useContext(ThemeContext);
  const themes = ['Light', 'Dark', 'System'];

  return (
    <View style={[styles.container, { backgroundColor: colors.background, padding: 20 }]}>
      <Text style={{ color: colors.text, marginBottom: 10 }}>Theme</Text>
      <View style={styles.row}>
        {themes.map(t => (
          <TouchableOpacity 
            key={t} 
            onPress={() => setThemeMode(t.toLowerCase())}
            style={[styles.themeBtn, { backgroundColor: themeMode === t.toLowerCase() ? colors.primary : colors.card }]}
          >
            <Text style={{ color: themeMode === t.toLowerCase() ? 'white' : colors.text }}>{t}</Text>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
}

// --- 3. NAVIGATION ---
const Tab = createBottomTabNavigator();
const Stack = createNativeStackNavigator();

function HomeTabs() {
  return (
    <Tab.Navigator screenOptions={{ headerShown: false }}>
      <Tab.Screen name="Home" component={HomeScreen} options={{ tabBarIcon: ({color}) => <Ionicons name="home" size={20} color={color}/> }} />
      <Tab.Screen name="Reports" component={View} options={{ tabBarIcon: ({color}) => <Ionicons name="document-text" size={20} color={color}/> }} />
      <Tab.Screen name="Settings" component={SettingsScreen} options={{ tabBarIcon: ({color}) => <Ionicons name="settings" size={20} color={color}/> }} />
    </Tab.Navigator>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <NavigationContainer>
        <Stack.Navigator>
          <Stack.Screen name="Main" component={HomeTabs} options={{ headerShown: false }} />
          <Stack.Screen name="ReportAnIssue" component={ReportAnIssue} title="Report an Issue" />
        </Stack.Navigator>
      </NavigationContainer>
    </ThemeProvider>
  );
}

// --- 4. STYLES ---
const styles = StyleSheet.create({
  container: { flex: 1, paddingHorizontal: 15 },
  header: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', paddingVertical: 20 },
  title: { fontSize: 24, fontWeight: 'bold' },
  mainBtn: { padding: 15, borderRadius: 10, backgroundColor: '#E1E1E1', alignItems: 'center', marginVertical: 10, flexDirection: 'row', justifyContent: 'center' },
  btnText: { fontWeight: '600' },
  errorBox: { marginTop: 20, padding: 15, borderRadius: 8, borderLeftWidth: 4, borderLeftColor: 'red', backgroundColor: '#FFF5F5' },
  uploadBox: { height: 180, borderRadius: 15, borderStyle: 'dashed', borderWidth: 1, borderColor: '#ccc', justifyContent: 'center', alignItems: 'center', marginVertical: 20 },
  recordBtn: { padding: 12, borderRadius: 8, marginTop: 10, alignItems: 'center' },
  input: { height: 120, borderRadius: 10, padding: 15, marginVertical: 20, textAlignVertical: 'top' },
  row: { flexDirection: 'row', justifyContent: 'space-between' },
  themeBtn: { padding: 10, borderRadius: 20, width: '30%', alignItems: 'center' }
});
