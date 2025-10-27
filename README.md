Currently, I use a .env file with a static API base URL like http://192.168.x.x/my_api, but my PC’s IP address changes frequently (dynamic IP). Each time, I have to rebuild the app to update the IP.

I want to add a feature in my Flutter app that allows me to change the API base URL directly inside the app without rebuilding.

Here’s what I want this feature to include:

1. A Settings page with a TextField to input the API Base URL (e.g. http://192.168.39.84/my_api).


2. A Save button that stores the entered URL using SharedPreferences.


3. The app should always use the stored API Base URL for all network requests.


4. If no custom URL is set, it should use a default fallback (like http://192.168.0.100/my_api).


5. Optional: show a “Saved successfully” message when the URL is saved.


6. Optional: allow changing the URL anytime from the settings menu.



Please provide:

A reusable ApiConfig class for getting and setting the stored base URL.

A simple SettingsPage UI with input and save functionality.

Example of how to use this dynamic base URL in my API calls (like in a login request using http package).


Bonus Tips

You can make it even better by auto-detecting if the saved IP works, and show a green “Connected” or red “Failed” status.

You can also use a TextField with only the IP and let the rest of the URL be auto-filled.

Make a file like lib/config/api_config.dart:

import 'package:shared_preferences/shared_preferences.dart';

class ApiConfig {
  static const String _key = "api_base_url";
  static const String _defaultUrl = "http://192.168.0.100/malawi_police_traffic";

  static Future<String> getBaseUrl() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_key) ?? _defaultUrl;
  }

  static Future<void> setBaseUrl(String url) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_key, url);
  }
}


---

3. Create a simple Settings page

Example: lib/screens/settings_page.dart

import 'package:flutter/material.dart';
import '../config/api_config.dart';

class SettingsPage extends StatefulWidget {
  const SettingsPage({super.key});

  @override
  State<SettingsPage> createState() => _SettingsPageState();
}

class _SettingsPageState extends State<SettingsPage> {
  final TextEditingController _controller = TextEditingController();
  String currentUrl = '';

  @override
  void initState() {
    super.initState();
    _loadUrl();
  }

  void _loadUrl() async {
    String url = await ApiConfig.getBaseUrl();
    setState(() {
      currentUrl = url;
      _controller.text = url;
    });
  }

  void _saveUrl() async {
    await ApiConfig.setBaseUrl(_controller.text);
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('API URL saved! Restart app to apply.')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('API Settings')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'API Base URL',
                hintText: 'http://192.168.x.x/malawi_police_traffic',
              ),
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: _saveUrl,
              child: const Text('Save'),
            ),
          ],
        ),
      ),
    );
  }
}


---

4. Use it in your API calls

Wherever you call your backend (e.g. in login.dart):

import '../config/api_config.dart';
import 'package:http/http.dart' as http;

Future<void> login(String email, String password) async {
  String baseUrl = await ApiConfig.getBaseUrl();
  final url = Uri.parse("$baseUrl/auth/login.php");

  final response = await http.post(url, body: {
    "email": email,
    "password": password,
  });

  print("Login Response: ${response.body}");
}


---

5. 🎯 Usage Flow

1. Open your app.


2. Go to your “Settings” page.


3. Type your PC’s IP address (e.g. http://192.168.39.84/malawi_police_traffic).


4. Save it.


5. Restart the app (or call getBaseUrl() again dynamically).


6. Boom — your app connects with the new backend!




---

✅ Bonus Tips

You can make it even better by auto-detecting if the saved IP works, and show a green “Connected” or red “Failed” status.

You can also use a TextField with only the IP and let the rest of the URL be auto-filled.



---

Would you like me to show you a full working Flutter example (Settings page + login API call using saved IP)?

