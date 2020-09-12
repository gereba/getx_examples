---
title: Tadas-Todo GetX Example
---

2020-09-05

dev/flutter_demos/getx_examples/todo_app

 

Pseudo-Logic
============

 

main
----

Widget build( context) {

return GetMaterialApp(

-   initialBinding: **AuthBinding**() extends Bindings

    -   .dependencies() {

        -   **Get.put**\<*AuthController*\>(AuthController(), permanent: *true*)

-   home: Root()

     

Root utils
----------

Root extends **GetWidget**\<*AuthController*\> {

Widget build( context) {

return **GetX**(

-   initState: (_) async {

    -   Get.put\<UserController\>(UserController() )

-   builder: (_) {

    -   if (**Get.find**\<*AuthController*\>().user?.uid != null) {

        -   return *Home*()

    -   else {

        -   return *Login*()

 

AuthController 
---------------

**AuthController** extends **GetxController {**

FirebaseAuth \_auth = FirebaseAuth.instance

FirebaseUser \_firebaseUser = Rx\<FirebaseUser\>()

-   **onInit**() {

    -   \_firebaseUser get user =\> \_firebaseUser.value

-   **.createUser(name, email, password) async {**

    -   \_authResult = await \_auth.createUserWithEmailAndPassword(email,
        password)

    -   UserModel \_user = UserModel( id, name, email )  // with
        \_authResult.user

    -   if (await **Database.createNewUser**(_user) ) {

        -   **Get.find**\<*UserController*\>().user = \_user

        -   **Get.back**()

    -   else {

        -   snackbar(*Error creating Account)*

-   **.login(email, password) async {**

    -   \_authResult = await \_auth.signInWithEmailAndPassword(email, password)

    -   **Get.find**\<*UserController*\>().user = await
        Database().getUser(_authResult.user.uid)

    -   \_auth.signInWithEmailAndPassword()

    -   } catch(e) { snackbar(*Error signing in*)

-   .**signOut**() async {

    -   await \_auth.signOut()

    -   **Get.find**\<*UserController*\>().clear

    -   } catch(e) { snackbar(*Error signing out*)

 

UserController
--------------

**UserController** extends **GetxController**

Rx\<UserModel\> \_userModel = UserModel().**obs**

-   .get user =\> \_userModel.value

-   **.set** user(value) =\> this._userModel.value = value

-   **.clear**() { \_userModel.value = UserModel()

 

 

Login Screen
------------

>   [ Email ]

>   [ Password ]

>   ( Log In ) —\> **authController.login**( Email, Password )

>   ( Sign Up ) —\> [SignUp Screen](#SignUp)

 

**Login** extends **GetWidget**\<*AuthController*\>

final emailController

final passwordController

Widget build( context ) { return Scaffold(

-   body:  Column [

    -   TextFormField( "*Email*"

    -   TextFormField( "*Password*"

    -   RaisedButtom( "*Log In*" ),

        -   onPressed: () {

            -   **controller.login**( emailController.text,
                passwordController.text )

    -   FlatButton( "*Sign Up*"),

        -   onPressed: () {

            -   **Get.to**( SignUp() )

 

 

SignUp Screen
-------------

**SignUp** extends **GetWidget**\<*AuthController*\>

final nameController

final emailController

final passwordController

Widget build( context ) { return Scaffold(

-   body:  Column [

    -   TextFormField( "*Full Name*"

    -   TextFormField( "*Email*"

    -   TextFormField( "*Password*"

    -   FlatButton( "*Sign Up*"),

        -   onPressed: () {

            -   **controller.createUser**( nameController, emailController,
                passwordController)

            -   // if success authController **Get.back()** to Root() widget
                which checks if authController.user.uid is not null and return
                to Home() page.

 

Home Screen
-----------

**Home** extends **GetWidget**\<*AuthController*\>

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

 

 

### TodoModel

import cloud_firestore.dart

class **TodoModel** {

String content, String todoId, Timestamp dateCreated, bool done

TodoModel( this.content, this.todoId, this.dateCreated, this.done )

 

TodoModel.fromDocumentSnapshot({DocumentSnapshot docSnap}) {

-   todoId = docSnap.documentID

-   content = docSnap.data["content"]

-   dateCreated = docSnap.data["dateCreated"]

-   done = docSnap.data["done"]

 

### UserModel

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

 
