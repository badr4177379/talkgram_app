```dart
import 'package:flutter/material.dart';
import 'dart:async'; // للـ Timer

// --- نماذج البيانات (تبقى كما هي) ---
enum MessageStatus { sending, sent, delivered, read }
class ChatMessage {
  final String id; String text; String senderId; DateTime timestamp; MessageStatus status;
  final String? repliedToMessageId; final String? repliedToMessageText; final String? repliedToSenderId;
  ChatMessage({ required this.id, required this.text, required this.senderId, required this.timestamp, this.status = MessageStatus.sent, this.repliedToMessageId, this.repliedToMessageText, this.repliedToSenderId });
  String get formattedTime => "${timestamp.hour.toString().padLeft(2, '0')}:${timestamp.minute.toString().padLeft(2, '0')}";
  bool get isMe => senderId == 'me';
}
class ChatConversation {
  final String id; String contactName; String lastMessage; DateTime lastMessageTime; String avatarIdentifier; int unreadCount; bool isMuted;
  ChatConversation({ required this.id, required this.contactName, required this.lastMessage, required this.lastMessageTime, required this.avatarIdentifier, this.unreadCount = 0, this.isMuted = false });
  String get formattedLastMessageTime {
    final now = DateTime.now();
    if (now.difference(lastMessageTime).inDays == 0) return "${lastMessageTime.hour.toString().padLeft(2, '0')}:${lastMessageTime.minute.toString().padLeft(2, '0')}";
    else if (now.difference(lastMessageTime).inDays == 1) return "Yesterday";
    else return "${lastMessageTime.day}/${lastMessageTime.month}/${lastMessageTime.year.toString().substring(2)}";
  }
}
enum StoryType { image, text }
class Story {
  final String id; StoryType type; String contentUrl; Duration duration; DateTime timestamp; String? textContent; Color? backgroundColor;
  Story({ required this.id, required this.type, required this.contentUrl, this.duration = const Duration(seconds: 5), required this.timestamp, this.textContent, this.backgroundColor });
}
class UserStatus {
  final String userId; String userName; String userAvatarUrl; List<Story> stories; DateTime lastUpdated; bool allStoriesViewed;
  UserStatus({ required this.userId, required this.userName, required this.userAvatarUrl, required this.stories, required this.lastUpdated, this.allStoriesViewed = false });
}

enum AuthAction { login, signUp }

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  static const Color talkgramPrimaryBlue = Color(0xFF0079C1);
  static const Color talkgramAccentBlue = Color(0xFF00B0FF);
  static const Color talkgramDarkBlue = Color(0xFF005A9E);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Talkgram',
      theme: ThemeData(
        primaryColor: talkgramPrimaryBlue,
        colorScheme: ColorScheme.fromSwatch(primarySwatch: Colors.blue).copyWith(
          primary: talkgramPrimaryBlue,
          secondary: talkgramAccentBlue,
          background: Colors.white,
          onBackground: Colors.black87,
          error: Colors.redAccent,
        ),
        appBarTheme: const AppBarTheme(
          backgroundColor: talkgramPrimaryBlue,
          foregroundColor: Colors.white,
          elevation: 0.7,
        ),
        tabBarTheme: const TabBarTheme(
          labelColor: Colors.white,
          unselectedLabelColor: Colors.white70,
          indicatorColor: Colors.white,
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            backgroundColor: talkgramAccentBlue,
            foregroundColor: Colors.white,
            padding: const EdgeInsets.symmetric(vertical: 12.0),
            textStyle: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
          ),
        ),
        inputDecorationTheme: InputDecorationTheme(
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(25.0),
            borderSide: BorderSide(color: Colors.grey.shade300),
          ),
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(25.0),
            borderSide: const BorderSide(color: talkgramPrimaryBlue, width: 2.0),
          ),
          contentPadding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 10.0),
           filled: true,
           fillColor: Colors.grey.shade100,
        ),
        floatingActionButtonTheme: const FloatingActionButtonThemeData(
          backgroundColor: talkgramAccentBlue,
          foregroundColor: Colors.white,
        ),
        dividerTheme: DividerThemeData(space: 1, thickness: 0.8, color: Colors.grey.shade300),
        listTileTheme: ListTileThemeData(
          iconColor: talkgramPrimaryBlue.withOpacity(0.8),
        )
      ),
      home: const LoginScreen(),
    );
  }
}

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});
  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _contactController = TextEditingController();
  bool _isLoading = false;

  @override
  void dispose() {
    _contactController.dispose();
    super.dispose();
  }

  bool _isEmail(String input) {
    if (input.isEmpty) return false;
    final emailRegExp = RegExp(r"^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+");
    return emailRegExp.hasMatch(input);
  }
  bool _isPhoneNumber(String input) {
    if (input.isEmpty) return false;
    final numericInput = input.replaceAll(RegExp(r'[^\d+]'), '');
    if (numericInput.startsWith('+')) {
      return numericInput.substring(1).replaceAll(RegExp(r'[^\d]'), '').length >= 6 && numericInput.substring(1).replaceAll(RegExp(r'[^\d]'), '').length <= 14;
    }
    return numericInput.replaceAll(RegExp(r'[^\d]'), '').length >= 7 && numericInput.replaceAll(RegExp(r'[^\d]'), '').length <= 15;
  }

  Future<void> _proceedToOtp() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      final contactInfo = _contactController.text.trim();
      await Future.delayed(const Duration(seconds: 1));
      print('OTP (simulated) sent to: $contactInfo for LOGIN');
      if (mounted) setState(() => _isLoading = false);

      if (mounted) {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => OtpVerificationScreen(
              contactInfo: contactInfo,
              authAction: AuthAction.login,
            ),
          ),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24.0),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  Text('تسجيل الدخول إلى Talkgram', textAlign: TextAlign.center, style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                  const SizedBox(height: 30),
                  TextFormField(
                    controller: _contactController,
                    decoration: const InputDecoration(
                        labelText: 'البريد الإلكتروني أو رقم الهاتف',
                        prefixIcon: Icon(Icons.person_outline)),
                    keyboardType: TextInputType.text,
                    validator: (v) {
                      final value = v?.trim() ?? '';
                      if (value.isEmpty) return 'الرجاء إدخال البريد الإلكتروني أو رقم الهاتف';
                      if (!_isEmail(value) && !_isPhoneNumber(value)) return 'البريد الإلكتروني أو رقم الهاتف غير صحيح';
                      return null;
                    },
                  ),
                  const SizedBox(height: 25),
                  _isLoading
                      ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor))
                      : ElevatedButton(onPressed: _proceedToOtp, child: const Text('التالي')),
                  const SizedBox(height: 20),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      const Text('ليس لديك حساب؟ '),
                      TextButton(
                        onPressed: _isLoading ? null : () => Navigator.push(context, MaterialPageRoute(builder: (context) => const SignUpScreen())),
                        child: Text('إنشاء حساب جديد', style: TextStyle(fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class SignUpScreen extends StatefulWidget {
  const SignUpScreen({super.key});
  @override
  State<SignUpScreen> createState() => _SignUpScreenState();
}

class _SignUpScreenState extends State<SignUpScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _contactController = TextEditingController();
  bool _isLoading = false;

  @override
  void dispose() {
    _nameController.dispose();
    _contactController.dispose();
    super.dispose();
  }

  bool _isEmail(String input) {
    if (input.isEmpty) return false;
    final emailRegExp = RegExp(r"^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+");
    return emailRegExp.hasMatch(input);
  }
  bool _isPhoneNumber(String input) {
    if (input.isEmpty) return false;
    final numericInput = input.replaceAll(RegExp(r'[^\d+]'), '');
    if (numericInput.startsWith('+')) {
      return numericInput.substring(1).replaceAll(RegExp(r'[^\d]'), '').length >= 6 && numericInput.substring(1).replaceAll(RegExp(r'[^\d]'), '').length <= 14;
    }
    return numericInput.replaceAll(RegExp(r'[^\d]'), '').length >= 7 && numericInput.replaceAll(RegExp(r'[^\d]'), '').length <= 15;
  }

  Future<void> _proceedToOtp() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      final name = _nameController.text.trim();
      final contactInfo = _contactController.text.trim();

      await Future.delayed(const Duration(seconds: 1));
      print('OTP (simulated) sent to: $contactInfo for SIGN UP (Name: $name)');
      if (mounted) setState(() => _isLoading = false);

      if (mounted) {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => OtpVerificationScreen(
              contactInfo: contactInfo,
              authAction: AuthAction.signUp,
              name: name.isNotEmpty ? name : null,
            ),
          ),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('إنشاء حساب جديد')),
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24.0),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  Text('أخبرنا عنك قليلاً', textAlign: TextAlign.center, style: TextStyle(fontSize: 26, fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                  const SizedBox(height: 30),
                  TextFormField(
                    controller: _nameController,
                    decoration: const InputDecoration(labelText: 'الاسم الكامل (اختياري)', prefixIcon: Icon(Icons.person_outline)),
                    keyboardType: TextInputType.name,
                  ),
                  const SizedBox(height: 20),
                  TextFormField(
                    controller: _contactController,
                    decoration: const InputDecoration(
                        labelText: 'البريد الإلكتروني أو رقم الهاتف',
                        prefixIcon: Icon(Icons.contact_mail_outlined)),
                    keyboardType: TextInputType.text,
                    validator: (v) {
                      final value = v?.trim() ?? '';
                      if (value.isEmpty) return 'الرجاء إدخال البريد الإلكتروني أو رقم الهاتف';
                      if (!_isEmail(value) && !_isPhoneNumber(value)) return 'البريد الإلكتروني أو رقم الهاتف غير صحيح';
                      return null;
                    },
                  ),
                  const SizedBox(height: 30),
                  _isLoading
                      ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor))
                      : ElevatedButton(onPressed: _proceedToOtp, child: const Text('التالي')),
                  const SizedBox(height: 20),
                   Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      const Text('لديك حساب بالفعل؟ '),
                      TextButton(
                        onPressed: _isLoading ? null : () {
                           if (Navigator.canPop(context)) {Navigator.pop(context);} else { Navigator.pushReplacement(context, MaterialPageRoute(builder: (context) => const LoginScreen()));}
                        },
                        child: Text('تسجيل الدخول', style: TextStyle(fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class OtpVerificationScreen extends StatefulWidget {
  final String contactInfo;
  final AuthAction authAction;
  final String? name;

  const OtpVerificationScreen({
    super.key,
    required this.contactInfo,
    required this.authAction,
    this.name,
  });

  @override
  State<OtpVerificationScreen> createState() => _OtpVerificationScreenState();
}

class _OtpVerificationScreenState extends State<OtpVerificationScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _otpController = TextEditingController();
  bool _isLoading = false;
  int _countdown = 60;
  Timer? _timer;
  bool _canResend = false;

  @override
  void initState() {
    super.initState();
    startTimer();
  }

  void startTimer() {
    _canResend = false;
    _countdown = 60;
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      if (!mounted) {
        timer.cancel();
        return;
      }
      if (_countdown > 0) {
        if (mounted) setState(() => _countdown--);
      } else {
        timer.cancel();
        if (mounted) setState(() => _canResend = true);
      }
    });
  }

  @override
  void dispose() {
    _otpController.dispose();
    _timer?.cancel();
    super.dispose();
  }

  Future<void> _verifyOtp() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      final otp = _otpController.text.trim();
      await Future.delayed(const Duration(seconds: 1));
      print('OTP entered: $otp for ${widget.contactInfo}');

      if (otp == "123456") { // Simulate correct OTP
        if (mounted) setState(() => _isLoading = false);
        if (mounted) {
           ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم التحقق بنجاح!')));
          if (widget.authAction == AuthAction.login) {
            Navigator.pushReplacement(
              context,
              MaterialPageRoute(
                builder: (context) => EnterPasswordScreen(contactInfo: widget.contactInfo),
              ),
            );
          } else {
            Navigator.pushReplacement(
              context,
              MaterialPageRoute(
                builder: (context) => SetPasswordScreen(
                  contactInfo: widget.contactInfo,
                  name: widget.name,
                ),
              ),
            );
          }
        }
      } else {
        if (mounted) setState(() => _isLoading = false);
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(SnackBar(
            content: const Text('كود التحقق غير صحيح. حاول مرة أخرى.'),
            backgroundColor: Theme.of(context).colorScheme.error,
          ));
        }
      }
    }
  }

  Future<void> _resendOtp() async {
    if (!_canResend || !mounted) return;
    setState(() => _isLoading = true);
    await Future.delayed(const Duration(seconds: 1));
    print('OTP (simulated) RESENT to: ${widget.contactInfo}');
    if (mounted) {
      setState(() {
        _isLoading = false;
        startTimer();
      });
      ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم إعادة إرسال كود التحقق (وهمياً).')));
    }
  }


  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('التحقق من الرمز')),
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24.0),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  Text(
                    'تم إرسال كود التحقق إلى:',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 18, color: Theme.of(context).primaryColor),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    widget.contactInfo,
                    textAlign: TextAlign.center,
                    style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 30),
                  TextFormField(
                    controller: _otpController,
                    decoration: const InputDecoration(
                      labelText: 'كود التحقق (مثال: 123456)',
                      prefixIcon: Icon(Icons.password_outlined),
                    ),
                    keyboardType: TextInputType.number,
                    textAlign: TextAlign.center,
                    maxLength: 6,
                    validator: (v) {
                      if (v == null || v.isEmpty) return 'الرجاء إدخال كود التحقق';
                      if (v.length < 6) return 'الكود يجب أن يكون 6 أرقام';
                      return null;
                    },
                  ),
                  const SizedBox(height: 25),
                  _isLoading
                      ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor))
                      : ElevatedButton(onPressed: _verifyOtp, child: const Text('تحقق')),
                  const SizedBox(height: 20),
                  TextButton(
                    onPressed: _canResend ? _resendOtp : null,
                    child: Text(
                      _canResend ? 'إعادة إرسال الكود' : 'إعادة الإرسال بعد $_countdown ثانية',
                      style: TextStyle(color: _canResend ? Theme.of(context).primaryColor : Colors.grey),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class EnterPasswordScreen extends StatefulWidget {
  final String contactInfo;
  const EnterPasswordScreen({super.key, required this.contactInfo});

  @override
  State<EnterPasswordScreen> createState() => _EnterPasswordScreenState();
}

class _EnterPasswordScreenState extends State<EnterPasswordScreen> {
  final _formKey = GlobalK
