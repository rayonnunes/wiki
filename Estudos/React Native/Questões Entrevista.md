# Elementos básicos do [[React Native]]

- View (android: ViewGroup; iOS: UIView; web: div)
- Text (android: TextView; iOS: UITextView; web: p)
- TextInput
- Image
- ScrollView
- Touchable/Pressable
- Switch
- Alert
- TextInput
- StatusBar
- StyleSheet
- FlatList

# Elementos específicos do android:
- BackHandler - detect hardware button presses for back navigation
- PermissionsAndroid - Acess permissions model introduced in android M
- ToastAndroid

# Elementos específicos do iOS:
- ActionSheetIOS - Displays native to iOS [Action Sheet](https://developer.apple.com/design/human-interface-guidelines/ios/views/action-sheets/) component 

# Image
```jsx
 <Image
  source={{
	uri: 'https://reactnative.dev/docs/assets/p_cat2.png',
  }}
  style={{width: 200, height: 200}}
/>
```

# TextInput
```jsx
<TextInput
	style={{height: 40}}
	placeholder="Type here to translate!"
	onChangeText={newText => setText(newText)}
	defaultValue={text}
  />
```

# Platform-Specific code
```jsx
import {Platform, StyleSheet} from 'react-native';  
  
const styles = StyleSheet.create({  
	height: Platform.OS === 'ios' ? 200 : 100,
	container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red',
      },
      android: {
        backgroundColor: 'green',
      },
      default: {
        // other platforms, web for example
        backgroundColor: 'blue',
      },
    }),
  },
});

const Component = Platform.select({  
	ios: () => require('ComponentIOS'),  
	android: () => require('ComponentAndroid'),
	native: () => require('ComponentForNative'),  
	default: () => require('ComponentForWeb'),
})();
```
Separados por arquivo, exemplo:
`BigButton.ios.js`
`BigButton.android.js`

```js
import BigButton from './BigButton';
```


# Style
```jsx
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const LotsOfStyles = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.red}>just red</Text>
      <Text style={styles.bigBlue}>just bigBlue</Text>
      <Text style={[styles.bigBlue, styles.red]}>bigBlue, then red</Text>
      <Text style={[styles.red, styles.bigBlue]}>red, then bigBlue</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginTop: 50,
  },
  bigBlue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

export default LotsOfStyles;
```