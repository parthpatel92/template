```dart


import 'dart:convert';
import 'dart:developer';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<UserModel> userList = [];
  bool isSearchMode = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('User\'s Profile'),
        actions: [
          IconButton(
            onPressed: () async {
              UserModel? user = await showDialog(
                context: context,
                builder: (context) => UserProfileDialog(),
              );
              setState(() {
                if (user != null) {
                  log(user.toString());
                  userList.add(user);
                }
              });
            },
            icon: Icon(Icons.add_box),
          ),
        ],
      ),
      body: userList.isEmpty
          ? Center(
              child: Text('No Data'),
            )
          : ListView.builder(
              itemCount: userList.length,
              itemBuilder: (context, index) {
                final data = userList[index];
                return ListTile(
                  leading: CircleAvatar(
                    radius: 35,
                    child: data.profile != null
                        ? ClipOval(
                            child: Image.file(
                              File(data.profile!),
                              height: 70,
                              width: 60,
                              fit: BoxFit.fill,
                            ),
                          )
                        : Icon(Icons.person),
                  ),
                  title: Text(data.name),
                  subtitle: Text(data.email),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      InkWell(
                        onTap: () {
                          setState(() {
                            userList.removeAt(index);
                          });
                        },
                        child: Icon(
                          Icons.delete,
                          color: Colors.redAccent,
                        ),
                      ),
                      SizedBox(width: 10),
                      InkWell(
                        onTap: () async {
                          UserModel? updatedUser = await showDialog(
                            context: context,
                            builder: (context) => UserProfileDialog(
                              userToEdit: data,
                              isEditMode: true,
                            ),
                          );
                          if (updatedUser != null) {
                            setState(() {
                              userList[index] = updatedUser;
                            });
                          }
                        },
                        child: Icon(
                          Icons.edit,
                          color: Colors.blue,
                        ),
                      )
                    ],
                  ),
                );
              },
            ),
    );
  }
}

class UserProfileDialog extends StatefulWidget {
  final UserModel? userToEdit;
  final bool isEditMode;

  const UserProfileDialog({
    super.key,
    this.userToEdit,
    this.isEditMode = false,
  });

  @override
  State<UserProfileDialog> createState() => _UserProfileDialogState();
}

class _UserProfileDialogState extends State<UserProfileDialog> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _descriptionController = TextEditingController();

  String? _profile;

  @override
  void initState() {
    super.initState();
    // If in edit mode, populate the fields with existing data
    if (widget.isEditMode && widget.userToEdit != null) {
      _nameController.text = widget.userToEdit!.name;
      _emailController.text = widget.userToEdit!.email;
      _descriptionController.text = widget.userToEdit!.description ?? '';
      _profile = widget.userToEdit!.profile;
    }
  }

  pickImageFile() async {
    ImagePicker imagePicker = ImagePicker();
    final pickedFile = await imagePicker.pickImage(source: ImageSource.gallery);
    if (pickedFile != null) {
      setState(() {
        _profile = pickedFile.path;
      });
    }
  }

  onSave() {
    // Validate required fields
    if (_nameController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Please enter a name')),
      );
      return;
    }

    if (_emailController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Please enter an email')),
      );
      return;
    }

    UserModel user = UserModel(
      name: _nameController.text.trim(),
      email: _emailController.text.trim(),
      description: _descriptionController.text.trim().isEmpty ? null : _descriptionController.text.trim(),
      profile: _profile,
    );
    Navigator.pop(context, user);
  }

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Theme(
      data: ThemeData(useMaterial3: false),
      child: AlertDialog(
        contentPadding: EdgeInsets.zero,
        titlePadding: EdgeInsets.all(15),
        title: Center(child: Text(widget.isEditMode ? 'Edit User Profile' : 'Add User Profile')),
        content: Container(
          decoration: BoxDecoration(border: Border(top: BorderSide())),
          width: MediaQuery.of(context).size.width,
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              SizedBox(height: 10),
              Center(
                child: GestureDetector(
                  onTap: pickImageFile,
                  child: CircleAvatar(
                    radius: 35,
                    backgroundColor: Colors.grey.shade100,
                    child: _profile != null
                        ? ClipOval(
                            child: Image.file(
                              File(_profile!),
                              width: 70,
                              height: 70,
                              fit: BoxFit.cover,
                            ),
                          )
                        : Icon(Icons.person, size: 40),
                  ),
                ),
              ),
              SizedBox(height: 10),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: TextField(
                  controller: _nameController,
                  decoration: InputDecoration(
                    filled: true,
                    border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
                    contentPadding: EdgeInsets.symmetric(horizontal: 12),
                    label: Text('User Name'),
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: TextField(
                  controller: _emailController,
                  keyboardType: TextInputType.emailAddress,
                  decoration: InputDecoration(
                    filled: true,
                    border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
                    contentPadding: EdgeInsets.symmetric(horizontal: 12),
                    label: Text('Email'),
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: TextField(
                  maxLines: 3,
                  minLines: 1,
                  controller: _descriptionController,
                  decoration: InputDecoration(
                    filled: true,
                    border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
                    contentPadding: EdgeInsets.symmetric(horizontal: 12),
                    label: Text('Description'),
                  ),
                ),
              ),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () {
              Navigator.pop(context);
            },
            child: Text('   Cancel   '),
          ),
          ElevatedButton(
            onPressed: onSave,
            child: Text(widget.isEditMode ? '   Update   ' : '   Save   '),
          )
        ],
      ),
    );
  }
}

class UserModel {
  final String name;
  final String email;
  final String? profile;
  final String? description;

  UserModel({
    required this.name,
    required this.email,
    this.profile,
    this.description,
  });

  UserModel copyWith({
    String? name,
    String? email,
    String? profile,
    String? description,
  }) {
    return UserModel(
      name: name ?? this.name,
      email: email ?? this.email,
      profile: profile ?? this.profile,
      description: description ?? this.description,
    );
  }

  Map<String, dynamic> toMap() {
    return <String, dynamic>{
      'name': name,
      'email': email,
      'profile': profile,
      'description': description,
    };
  }

  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      name: map['name'] as String,
      email: map['email'] as String,
      profile: map['profile'] != null ? map['profile'] as String : null,
      description: map['description'] != null ? map['description'] as String : null,
    );
  }

  String toJson() => json.encode(toMap());

  factory UserModel.fromJson(String source) => UserModel.fromMap(json.decode(source) as Map<String, dynamic>);

  @override
  String toString() {
    return 'UserModel(name: $name, email: $email, profile: $profile, description: $description)';
  }

  @override
  bool operator ==(covariant UserModel other) {
    if (identical(this, other)) return true;

    return other.name == name && other.email == email && other.profile == profile && other.description == description;
  }

  @override
  int get hashCode {
    return name.hashCode ^ email.hashCode ^ profile.hashCode ^ description.hashCode;
  }
}



```
