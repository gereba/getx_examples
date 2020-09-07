---
title: Tadas-Todo GetX Example
---

2020-09-05

dev/flutter_demos/getx_examples/todo_app

 

Pseudo-Logic
============

 

main
----

-   **AuthBinding** extends Bindings

    -   .dependencies()

        -   AuthController

-   Root

    -   **AuthController** extends GetxController

        -   FirebaseAuth \_auth

        -   FirebaseUser

        -   **.createUser(name, email, password)**

            -   UserModel

            -   **Database.createNewUser**(user)

            -   UserController.user

            -   snackbar(*Error creating Account)*

        -   **.login(email, password)**

            -   \_auth.signInWithEmailAndPassword()

            -   Database.getUser(id)

            -   UserController.user

            -   snackbar(*Error signing in*)

        -   .**signOut**()

            -   \_auth.signOut()

            -   UserController.clear

            -   snackbar(*Error signing out*)

    -   **UserController** extends GetxController

        -   Rx\<UserModel\> \_userModel = UserModel().obs

        -   **.set** user(value)

        -   **.clear()**

    -   **Home**()

    -   **Login** extends GetWidget\<AuthController\>

        -   final emailController

        -   final passwordController

    -   **SignUp** extends GetWidget\<AuthController\>

        -   final nameController

        -   final emailController

        -   final passwordController

        -    

 

home
----

**Home** extends GetWidget\<AuthController\>

final \_todoController

Widget build() {return Scafold( appbar: AppBar(

-   title: **GetX\<UserController\>(**

    -   initState: (_) *async* {

        -   **Get.find\<UserController\>().user** = *await*

            -   **Database().getUser(Get.find\<AuthController\>().user.uid) **

    -   builder: (_) {

        -   if (_.user.name != null) { return Text('*Welcome* ' + \_.user.name )

        -   else { return Text('*loading*…' )

-   actions: [

    -   IconButton( icon: Icon(Icons.exit_to_app),

        -   onPressed: () {

            -   **controller.signOut**()

    -   IconButton( icon: Icon(Icons.edit),

        -   onPressed: () {

            -   **Get.isDarkMode**

                -   ? Get.changeTheme( ThemeData.light())

                -   : Get.changeTheme( ThemeData.dark())

-   body: Column( children: \<Widget\> [

    -   Text( "*Add Todo Here:*" )

    -   Card(child: Padding( child: Row( children: [

        -   TextFormField(),

        -   IconButton( icon: Icons.add),

            -   onPressed: () {

                -   **Database().addTodo**( \_todoController.text,
                    controller.user.uid );

                -   **\_todoController.clear**()

    -   Text( "*Your Todos*" )

    -   **GetX\<TodoController\>(**

        -   init: **Get.put\<TodoController\>(TodoController(**) )

        -   builder: (**TodoController todoCtr**) {

            -   if (todoCtr != null && todoCtr.todos != null) {

                -   return Expanded( child: ListView.builder(

                -   itemCount: todoCtr.todos.length,

                -   itemBuilder: ( \_ , index ) {

                -   return TodoCard(

                -   uid: controller.user.uid,

                -   todo: todoCtr.todos[index] );

            -   else { Text( "*loading*…" )

 

 

todoController
--------------

**TodoController** extends GetxController

Rx\<List\<TodoModel\>\> todoList = Rx\<List\<TodoModel\>\>();

List\<TodoModel\> get todos =\> todoList.value;

 

\@override

void onInit() {

-   String uid = **Get.find\<AuthController\>().user.uid**;

-   todoList.bindStream( Database().todoStream(uid) );  // stream coming from
    firebase

 

 

### models/todo.dart

import cloud_firestore.dart

class **TodoModel** {

String content, String todoId, Timestamp dateCreated, bool done

TodoModel( this.content, this.todoId, this.dateCreated, this.done )

 

TodoModel.fromDocumentSnapshot({DocumentSnapshot docSnap}) {

-   todoId = docSnap.documentID

-   content = docSnap.data["content"]

-   dateCreated = docSnap.data["dateCreated"]

-   done = docSnap.data["done"]

 

### models/user.dart

import cloud_firestore.dart

class **UserModel** {

String id, name. email

UserModel( {this.id, this.name, this.email} )

 

UserModel.fromDocumentSnapshot({DocumentSnapshot docSnap}) {

-   id = docSnap.documentID

-   name = docSnap["name"]

-   email = docSnap["email’]

 

Database
--------

class **Database**

final Firestore \_firestore = Firestore.instance;

 

Future\<bool\> **createNewUser**(UserModel user) async {

try {

-   await \_firestore.collection("users").document(user.id).setData( {

    -   "name": user.name

    -   "email": user.email

-   return true

} catch (e) { print(e); return false;} }

 

Future\<UserModel\> **getUser**(String uid) async {

try {

-   DocumentSnapshot \_doc = await
    \_firestore.collection("users").document(uid).get();

-   return UserMode.fromDocumentSnapshot(documentSnapshot: \_doc);

} catch (e) { print(e); **rethrow**; }

 

 

Future\<void\> **addTodo**(String content, String uid) async {

try {

-   await \_firestore.collection("users").document(uid)

    -   .collection("todos").document(todoId)

    -   .add( {"content": content, "dateCreated": Timestamp.now() , "done":
        false});

} catch (e)  { print(e); **rethrow**; }

 

Future\<void\> **updateTodo**(bool newValue, String uid, String todoId) async {

try {

-   \_firestore.collection("users").document(uid)

    -   .collection("todos").document(todoId)

    -   .updateData( {"done": newValue});

} catch (e)  { print(e); **rethrow**; }

 

 

Stream\<List\<TodoModel\>\> **todoStream**(String uid) {

return \_firestore

.collection("users")

.document(uid)

.collection("todos")

.orderBy("dateCreated", descending: true)

.snapshots()

.*map*( (QuerySnapshot query) {

-   List\<TodoModel\> retVal = List();

-   query.documents.forEach( (element) {

    -   retVal.add(TodoModel.fromDocumentSnapshot(element));

    -   });

-   return retVal;

-   });

 
