# staj1
Mobil Menü Uygulaması

firebase_authantication.dart :
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_training/model/model_customer.dart';
import '../../model/model_waiter.dart';
class FirebaseAuthentication {
FirebaseAuthentication._init();
static final FirebaseAuthentication instance = FirebaseAuthentication._init();
final collection = FirebaseFirestore.instance.collection("Accounts");
Future<dynamic> login({
required String email,
required String password,
}) async {
final snapshot = await collection
.where("email", isEqualTo: email)
.where("password", isEqualTo: password)
.get();
if (snapshot.docs.isEmpty) {
return null;
}
final doc = snapshot.docs.first;
final accountType = doc.get("accountType");
final accountModelMap = {
"customer": CustomerModel.fromSnapshot(doc),
"waiter": WaiterModel.fromSnapshot(doc),
};
return accountModelMap[accountType];
}
}
customer_model.dart :
class CustomerModel {
final String? id;
final String? accountType;
final String? name;
final String? surname;
final String? email;
final String? imageUrl;
final String? phone;
CustomerModel({
this.id,
this.accountType,
this.name,
this.surname,
this.email,
this.imageUrl,
this.phone,
});
factory CustomerModel.fromSnapshot(snapshot) {
final data = snapshot.data();
return CustomerModel(
id: snapshot.id,
accountType: data["accountType"],
name: data["name"],
surname: data["surname"],
email: data["email"],
imageUrl: data["imageUrl"],
phone: data["phone"],
);
}
factory CustomerModel.fromMap(map) {
return CustomerModel(
id: map["id"],
accountType: map["accountType"],
name: map["name"],
surname: map["surname"],
email: map["email"],
imageUrl: map["imageUrl"],
phone: map["phone"],
);
}
Map<String, dynamic> toMap() => {
"accountType": accountType,
"name": name,
"surname": surname,
"email": email,
"imageUrl": imageUrl,
"phone": phone,
};
}
screen_waiter_password_change.dart :
import 'package:firebase_training/components/buttons/button_action.dart';
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import '../classes/class_snackbar.dart';
import '../classes/class_validation.dart';
import '../components/scroll_view.dart';
import '../components/text/text.dart';
import '../components/text/text_field.dart';
import '../constants/constants_color.dart';
import '../controllers/controller_main.dart';
import 'screen_login.dart';
class WaiterPasswordChangeScreen extends StatelessWidget {
WaiterPasswordChangeScreen({super.key});
final validationClass = ValidationClass.instance;
final snackbarClass = SnackbarClass.instance;
final mainController = MainController.instance;
// final firebaseUser = FirebaseUser.instance;
final tecCurrentPassword = TextEditingController();
final tecNewPassword = TextEditingController();
final tecNewPasswordAgain = TextEditingController();
@override
Widget build(BuildContext context) {
return Scaffold(
appBar: AppBar(
title: const MyText(
text: "Şifre Değiştir",
fontWeight: FontWeight.bold,
),
centerTitle: true,
),
body: MyScrollView(
children: [
Padding(
padding: const EdgeInsets.symmetric(vertical: 10),
child: MyText(
textAlign: TextAlign.start,
text: "Lütfen mevcut şifrenizi giriniz.",
color: uiGreenDark,
fontSize: 15,
fontWeight: FontWeight.bold,
),
),
MyTextField(
labelText: "Mevcut şifre",
controller: tecCurrentPassword,
icon: Icons.lock_outline,
obscureText: true,
),
Padding(
padding: const EdgeInsets.symmetric(vertical: 10),
child: MyText(
textAlign: TextAlign.start,
text: "Lütfen yeni şifrenizi giriniz.",
color: uiGreenDark,
fontSize: 15,
fontWeight: FontWeight.bold,
),
),
MyTextField(
labelText: "Yeni Şifre",
controller: tecNewPassword,
obscureText: true,
icon: Icons.lock_outline,
),
MyTextField(
labelText: "Yeni Şifre (Tekrar)",
controller: tecNewPasswordAgain,
obscureText: true,
icon: Icons.lock_outline,
),
Padding(
padding: const EdgeInsets.symmetric(vertical: 10),
child: SizedBox(
width: double.infinity,
child: MyActionButton(
onPressed: () => updatePasswordUser(context),
text: "Şifre Değiştir",
),
),
),
Padding(
padding: const EdgeInsets.symmetric(vertical: 10),
child: SizedBox(
width: double.infinity,
child: MyActionButton(
backgroundColor: uiGrey,
onPressed: () => Get.to(() => LoginScreen()),
text: "Hesabı Sil",
),
),
),
],
),
);
}
Future<void> updatePasswordUser(BuildContext context) async {
final userId = mainController.myAccount.id;
final currentPassword = tecCurrentPassword.text.trim();
final newPassword = tecNewPassword.text.trim();
final newPasswordAgain = tecNewPasswordAgain.text.trim();
final validateInputs = validationClass.validateInputs(
context: context,
inputList: [currentPassword, newPassword, newPasswordAgain],
);
if (!validateInputs) {
return;
}
}
final validatePassword = validationClass.validatePassword(
context: context,
password: newPassword,
);
if (!validatePassword) {
return;
}
final validatePasswords = validationClass.validatePasswords(
context: context,
password: newPassword,
passwordAgain: newPasswordAgain,
);
if (!validatePasswords) {
return;
}
final firebasePassword = await firebaseUser.getUserPasswordById(userId);
if (currentPassword != firebasePassword) {
// ignore: use_build_context_synchronously
snackbarClass.show(
context: context,
message: "Mevcut şifreyi yanlış girdiniz.",
icon: Icons.error,
);
return;
}
await firebaseUser.updateUserPassword(
userId: userId,
password: newPassword,
);
// ignore: use_build_context_synchronously
snackbarClass.show(
context: context,
message: "Yeni şifre bilgileri güncellendi.",
icon: Icons.check_circle,
backgroundColor: uiGreenDark,
);
Get.back();
}
}

screen_customer_qr_scanner.dart :
import 'dart:io';
import 'package:firebase_training/components/buttons/button_action.dart';
import 'package:firebase_training/components/scroll_view.dart';
import 'package:firebase_training/components/text/text_field.dart';
import 'package:firebase_training/screens/screen_customer_menu.dart';
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:qr_code_scanner/qr_code_scanner.dart';
import '../classes/class_snackbar.dart';
import '../classes/class_validation.dart';
import '../components/container.dart';
import '../components/text/text.dart';
import '../controllers/controller_main.dart';
class CustomerQrScannerScreen extends StatefulWidget {
const CustomerQrScannerScreen({super.key});
@override
State<CustomerQrScannerScreen> createState() =>
_CustomerQrScannerScreenState();
}
class _CustomerQrScannerScreenState extends State<CustomerQrScannerScreen> {
final mainController = MainController.instance;
final validationClass = ValidationClass.instance;
final snackbarClass = SnackbarClass.instance;
final tecCode = TextEditingController();
final GlobalKey qrKey = GlobalKey(debugLabel: 'QR');
Barcode? result;
QRViewController? controller;
@override
void initState() {
Future.delayed(const Duration(milliseconds: 100))
.whenComplete(() => controller?.resumeCamera());
super.initState();
}
@override
void reassemble() {
super.reassemble();
if (Platform.isAndroid) {
controller!.pauseCamera();
} else if (Platform.isIOS) {
controller!.resumeCamera();
}
}
void onQRViewCreated(QRViewController controller) {
this.controller = controller;
controller.scannedDataStream.listen((scanData) {
final code = scanData.code;
if (code != null) {
tecCode.text = code;
}
});
}
@override
void dispose() {
controller?.dispose();
super.dispose();
}
@override
Widget build(BuildContext context) {
final scanArea = (MediaQuery.of(context).size.width < 400 ||
MediaQuery.of(context).size.height < 400)
? 175.0
: 300.0;
return Scaffold(
appBar: AppBar(
centerTitle: true,
title: const MyText(
text: "Qr Kod",
fontWeight: FontWeight.bold,
),
),
body: MyScrollView(
crossAxisAlignment: CrossAxisAlignment.center,
children: [
MyContainer(
margin: const EdgeInsets.symmetric(vertical: 30),
dimension: 250,
child: QRView(
key: qrKey,
onQRViewCreated: onQRViewCreated,
overlay: QrScannerOverlayShape(
borderColor: Colors.white,
borderRadius: 10,
borderLength: 30,
borderWidth: 10,
cutOutSize: scanArea,
),
),
),
const MyText(
text: "QR Kod Tarayınız",
fontWeight: FontWeight.bold,
fontSize: 15,
color: Colors.grey,
),
const MyText(
text: "veya Masa Kodunu Giriniz.",
fontWeight: FontWeight.bold,
fontSize: 15,
color: Colors.grey,
),
MyTextField(
labelText: "Masa Kodunu Giriniz",
controller: tecCode,
textInputType: TextInputType.phone,
margin: const EdgeInsets.symmetric(vertical: 20),
icon: Icons.qr_code_outlined,
),
SizedBox(
width: double.infinity,
child: MyActionButton(
onPressed: () {
Get.to((CustomerMenuScreen()));
}, // => getUserByPhone(context),
text: "Onayla",
),
),
],
),
); }

firebase_order.dart :
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../model/model_order.dart';
class FirebaseOrder {
final collection = FirebaseFirestore.instance.collection("Orders");
Future<void> createOrder(OrderModel order) async {
await collection.doc(order.id).set({
"orderName": order.name,
"orderCount": order.count,
});
}
Future<OrderModel> getOrder(String orderId) async {
final snapshot = await collection.doc(orderId).get();
return OrderModel.fromSnapshot(snapshot);
}
Future<List<OrderModel>> getAllOrders(String orderName) async {
final snapshot =
await collection.where("orderName", isEqualTo: orderName).get();
return snapshot.docs.map((doc) => OrderModel.fromSnapshot(doc)).toList();
}
Future<void> deleteOrder(String orderId) async {
await collection.doc(orderId).delete();
}
Future<void> updateOrder(OrderModel order) async {
await collection.doc(order.id).update({
"orderName": order.name,
"orderCount": order.count,
});
}

