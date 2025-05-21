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
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _passwordController = TextEditingController();
  bool _obscurePassword = true;
  bool _isLoading = false;

  @override
  void dispose() {
    _passwordController.dispose();
    super.dispose();
  }

  Future<void> _loginWithPassword() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      // final password = _passwordController.text;
      await Future.delayed(const Duration(seconds: 1));
      print('Attempting login for ${widget.contactInfo} with password (hidden)');

      if (mounted) setState(() => _isLoading = false);
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم تسجيل الدخول بنجاح!')));
        Navigator.pushAndRemoveUntil(
          context,
          MaterialPageRoute(builder: (context) => const MainAppScreen()),
          (Route<dynamic> route) => false,
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('إدخال كلمة المرور')),
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
                  Text('مرحباً بعودتك!', textAlign: TextAlign.center, style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                  const SizedBox(height: 8),
                  Text('الرجاء إدخال كلمة المرور لحساب ${widget.contactInfo}', textAlign: TextAlign.center, style: const TextStyle(fontSize: 16)),
                  const SizedBox(height: 30),
                  TextFormField(
                    controller: _passwordController,
                    decoration: InputDecoration(
                      labelText: 'كلمة المرور',
                      prefixIcon: const Icon(Icons.lock_outline),
                      suffixIcon: IconButton(
                        icon: Icon(_obscurePassword ? Icons.visibility_off_outlined : Icons.visibility_outlined),
                        onPressed: () { if(mounted) setState(() => _obscurePassword = !_obscurePassword);},
                      ),
                    ),
                    obscureText: _obscurePassword,
                    validator: (v) {
                      if (v == null || v.isEmpty) return 'الرجاء إدخال كلمة المرور';
                      if (v.length < 6) return 'كلمة المرور قصيرة جداً';
                      return null;
                    },
                  ),
                  const SizedBox(height: 25),
                  _isLoading
                      ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor))
                      : ElevatedButton(onPressed: _loginWithPassword, child: const Text('تسجيل الدخول')),
                   const SizedBox(height: 10),
                   Align(alignment: Alignment.centerRight, child: TextButton(onPressed: _isLoading ? null : () => print('Forgot Password from EnterPasswordScreen?'), child: Text('نسيت كلمة المرور؟', style: TextStyle(color: Theme.of(context).primaryColor)))),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class SetPasswordScreen extends StatefulWidget {
  final String contactInfo;
  final String? name;
  const SetPasswordScreen({super.key, required this.contactInfo, this.name});

  @override
  State<SetPasswordScreen> createState() => _SetPasswordScreenState();
}

class _SetPasswordScreenState extends State<SetPasswordScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _confirmPasswordController = TextEditingController();
  bool _obscurePassword = true;
  bool _obscureConfirmPassword = true;
  bool _isLoading = false;

  @override
  void dispose() {
    _passwordController.dispose();
    _confirmPasswordController.dispose();
    super.dispose();
  }

  Future<void> _completeSignUp() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      // final password = _passwordController.text;
      await Future.delayed(const Duration(seconds: 1));
      print('Account creation attempt for ${widget.contactInfo} (Name: ${widget.name ?? 'N/A'}) with password (hidden)');

      if (mounted) setState(() => _isLoading = false);
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم إنشاء الحساب بنجاح!')));
        Navigator.pushAndRemoveUntil(
          context,
          MaterialPageRoute(builder: (context) => ProfileSetupScreen(initialName: widget.name)),
          (Route<dynamic> route) => false,
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('تعيين كلمة المرور')),
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
                  Text('إنشاء كلمة مرور قوية', textAlign: TextAlign.center, style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)),
                  const SizedBox(height: 8),
                  Text('لحساب ${widget.contactInfo}', textAlign: TextAlign.center, style: const TextStyle(fontSize: 16)),
                  const SizedBox(height: 30),
                  TextFormField(
                    controller: _passwordController,
                    decoration: InputDecoration(
                      labelText: 'كلمة المرور',
                      prefixIcon: const Icon(Icons.lock_outline),
                      suffixIcon: IconButton(
                        icon: Icon(_obscurePassword ? Icons.visibility_off_outlined : Icons.visibility_outlined),
                        onPressed: () { if(mounted) setState(() => _obscurePassword = !_obscurePassword);},
                      ),
                    ),
                    obscureText: _obscurePassword,
                    validator: (v) {
                      if (v == null || v.isEmpty) return 'الرجاء إدخال كلمة المرور';
                      if (v.length < 6) return 'كلمة المرور يجب أن تكون 6 أحرف على الأقل';
                      return null;
                    },
                  ),
                  const SizedBox(height: 20),
                  TextFormField(
                    controller: _confirmPasswordController,
                    decoration: InputDecoration(
                      labelText: 'تأكيد كلمة المرور',
                      prefixIcon: const Icon(Icons.lock_outline),
                       suffixIcon: IconButton(
                        icon: Icon(_obscureConfirmPassword ? Icons.visibility_off_outlined : Icons.visibility_outlined),
                        onPressed: () { if(mounted) setState(() => _obscureConfirmPassword = !_obscureConfirmPassword);},
                      ),
                    ),
                    obscureText: _obscureConfirmPassword,
                    validator: (v) {
                      if (v == null || v.isEmpty) return 'الرجاء تأكيد كلمة المرور';
                      if (v != _passwordController.text) return 'كلمتا المرور غير متطابقتين';
                      return null;
                    },
                  ),
                  const SizedBox(height: 25),
                  _isLoading
                      ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor))
                      : ElevatedButton(onPressed: _completeSignUp, child: const Text('إنشاء الحساب ومتابعة')),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class ProfileSetupScreen extends StatefulWidget {
  final String? initialName;
  const ProfileSetupScreen({super.key, this.initialName});
  @override
  State<ProfileSetupScreen> createState() => _ProfileSetupScreenState();
}
class _ProfileSetupScreenState extends State<ProfileSetupScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;
  bool _isLoading = false;
  bool _imageSelected = false;

 @override
  void initState() {
    super.initState();
    _nameController = TextEditingController(text: widget.initialName ?? '');
  }

  @override
  void dispose() {
    _nameController.dispose();
    super.dispose();
  }

  Future<void> _pickImage() async {
    if (mounted) {
      setState(() => _imageSelected = true);
      ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم اختيار صورة (وهمياً)!')));
    }
    print("Pick image function called - Image selected (theoretically)");
  }
  Future<void> _saveProfile() async {
    if (_formKey.currentState!.validate()) {
      if (mounted) setState(() => _isLoading = true);
      try {
        await Future.delayed(const Duration(seconds: 1));
        print('Profile Name Saved: ${_nameController.text.trim()}');
        if (_imageSelected) print('Profile Image was selected (theoretically).');
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم حفظ الملف الشخصي! ننتقل للرئيسية...')));
          Navigator.pushAndRemoveUntil(context, MaterialPageRoute(builder: (context) => const MainAppScreen()), (Route<dynamic> route) => false);
        }
      } catch (e) {
        if (mounted) ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('خطأ: ${e.toString()}'), backgroundColor: Theme.of(context).colorScheme.error));
      } finally {
        if (mounted) setState(() => _isLoading = false);
      }
    }
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(appBar: AppBar(title: const Text('إعداد الملف الشخصي')), body: SafeArea(child: Center(child: SingleChildScrollView(padding: const EdgeInsets.all(24.0), child: Form(key: _formKey, child: Column(mainAxisAlignment: MainAxisAlignment.center, crossAxisAlignment: CrossAxisAlignment.stretch, children: <Widget>[Text('أخبرنا عنك', textAlign: TextAlign.center, style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor)), const SizedBox(height: 30), Center(child: Stack(children: [CircleAvatar(radius: 60, backgroundColor: _imageSelected ? Theme.of(context).colorScheme.secondary.withOpacity(0.3) : Colors.grey.shade300, child: _imageSelected ? Icon(Icons.check_circle, size: 60, color: Theme.of(context).colorScheme.secondary) : Icon(Icons.person, size: 60, color: Colors.grey.shade700)), Positioned(bottom: 0, right: 0, child: CircleAvatar(radius: 20, backgroundColor: Theme.of(context).colorScheme.secondary, child: IconButton(icon: const Icon(Icons.camera_alt, color: Colors.white, size: 20), onPressed: _pickImage)))])), const SizedBox(height: 30), TextFormField(controller: _nameController, decoration: const InputDecoration(labelText: 'الاسم المعروض', prefixIcon: Icon(Icons.person_outline)), keyboardType: TextInputType.name, validator: (v) => (v == null || v.isEmpty) ? 'الرجاء إدخال اسمك' : (v.length < 3) ? 'الاسم قصير جداً' : null), const SizedBox(height: 40), _isLoading ? Center(child: CircularProgressIndicator(color: Theme.of(context).primaryColor)) : ElevatedButton(onPressed: _saveProfile, child: const Text('حفظ ومتابعة'))], ), ), ), ), ), );
  }
}

class MainAppScreen extends StatefulWidget {
  const MainAppScreen({super.key});
  @override
  State<MainAppScreen> createState() => MainAppScreenState();
}

class MainAppScreenState extends State<MainAppScreen> with SingleTickerProviderStateMixin {
  late TabController _tabController;
  int _currentTabIndex = 0;
  late List<ChatConversation> dummyConversations;
  bool _isSearching = false;
  final TextEditingController _searchController = TextEditingController();
  List<ChatConversation> _filteredConversations = [];

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this, initialIndex: 0);
    _tabController.addListener(() {
      if (mounted && (_tabController.indexIsChanging || _tabController.index != _currentTabIndex)) {
        setState(() => _currentTabIndex = _tabController.index);
      }
    });

    dummyConversations = List.generate(15, (index) {
      return ChatConversation(
        id: 'conv${index + 1}',
        contactName: "صديق ${index + 1}",
        lastMessage: "مرحباً! هذه رسالة تجريبية طويلة لترى كيف يظهر الـ ellipsis.",
        lastMessageTime: DateTime.now().subtract(Duration(minutes: index * 15)),
        avatarIdentifier: "ص${index + 1}",
        unreadCount: (index % 4 == 0) ? (index + 1) : 0,
      );
    });
    _filteredConversations = dummyConversations;
    _searchController.addListener(_filterConversations);
  }

  @override
  void dispose() {
    _tabController.dispose();
    _searchController.removeListener(_filterConversations);
    _searchController.dispose();
    super.dispose();
  }

  void _filterConversations() {
    final query = _searchController.text.toLowerCase();
    if (mounted) {
      setState(() {
        if (query.isEmpty) {
          _filteredConversations = dummyConversations;
        } else {
          _filteredConversations = dummyConversations
              .where((conversation) =>
                  conversation.contactName.toLowerCase().contains(query))
              .toList();
        }
      });
    }
  }

  Widget _buildFab() {
    switch (_currentTabIndex) {
      case 0:
        return FloatingActionButton(
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const SelectContactScreen(title: "New Chat")),
            );
          },
          child: const Icon(Icons.message),
        );
      case 1:
        return Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            FloatingActionButton(
              mini: true,
              onPressed: () => print("FAB for Text Status pressed"),
              child: const Icon(Icons.edit),
              heroTag: 'fab2',
            ),
            const SizedBox(height: 16),
            FloatingActionButton(
              onPressed: () => print("FAB for Camera Status pressed"),
              child: const Icon(Icons.camera_alt),
              heroTag: 'fab1',
            ),
          ],
        );
      case 2:
        return FloatingActionButton(
          onPressed: () => print("FAB for New Call pressed"),
          child: const Icon(Icons.add_call),
        );
      default:
        return Container();
    }
  }

  AppBar _buildAppBar() {
    if (_isSearching) {
      return AppBar(
        leading: IconButton(
            icon: const Icon(Icons.arrow_back),
            onPressed: () {
              if (mounted) {
                setState(() { _isSearching = false; _searchController.clear(); });
              }
            }),
        title: TextField(
            controller: _searchController,
            autofocus: true,
            decoration: const InputDecoration(
                hintText: 'ابحث...',
                hintStyle: TextStyle(color: Colors.white70),
                border: InputBorder.none,
                fillColor: Colors.transparent,
                filled: false,
                ),
            style: const TextStyle(color: Colors.white, fontSize: 18)),
        actions: [
          IconButton(
              icon: const Icon(Icons.clear),
              onPressed: () {
                if (mounted) _searchController.clear();
              })
        ],
        bottom: TabBar(
            controller: _tabController,
            tabs: const <Widget>[
              Tab(text: "CHATS"),
              Tab(text: "STATUS"),
              Tab(text: "CALLS")
            ]),
      );
    } else {
      return AppBar(
        title: const Text('Talkgram'),
        actions: <Widget>[
          IconButton(
              icon: const Icon(Icons.search),
              onPressed: () {
                if (mounted) setState(() => _isSearching = true);
              }),
          PopupMenuButton<String>(
            onSelected: (value) {
              print("Menu: $value");
              if (value == 'Settings') {
                Navigator.push(context, MaterialPageRoute(builder: (context) => const SettingsScreen()));
              } else if (value == 'New group') {
                 Navigator.push(context, MaterialPageRoute(builder: (context) => const SelectContactScreen(title: "New group")));
              }
            },
            itemBuilder: (BuildContext c) =>
                {'New group', 'Settings'}
                    .map((String choice) =>
                        PopupMenuItem<String>(value: choice, child: Text(choice)))
                    .toList(),
          ),
        ],
        bottom: TabBar(
            controller: _tabController,
            tabs: const <Widget>[
              Tab(text: "CHATS"),
              Tab(text: "STATUS"),
              Tab(text: "CALLS")
            ]),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildAppBar(),
      body: TabBarView(
        controller: _tabController,
        children: <Widget>[
          ChatsTabScreen(conversations: _filteredConversations),
          const StatusTabScreen(),
          const CallsTabScreen()
        ],
      ),
      floatingActionButton: _buildFab(),
    );
  }
}

class ChatsTabScreen extends StatefulWidget {
  final List<ChatConversation> conversations;
  const ChatsTabScreen({super.key, required this.conversations});

  @override
  State<ChatsTabScreen> createState() => ChatsTabScreenState();
}

class ChatsTabScreenState extends State<ChatsTabScreen> {
 @override
  Widget build(BuildContext context) {
    if (widget.conversations.isEmpty) {
      return const Center(child: Text("لا توجد دردشات تطابق بحثك أو لا توجد دردشات حالياً."));
    }
    return ListView.builder(
        itemCount: widget.conversations.length,
        itemBuilder: (context, index) {
          final conversation = widget.conversations[index];
          final hasUnread = conversation.unreadCount > 0;
          return ListTile(
            leading: CircleAvatar(
                backgroundColor: Theme.of(context).primaryColor.withOpacity(0.15),
                child: Text(conversation.avatarIdentifier, style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold))),
            title: Text(conversation.contactName, style: const TextStyle(fontWeight: FontWeight.bold)),
            subtitle: Text(conversation.lastMessage, maxLines: 1, overflow: TextOverflow.ellipsis),
            trailing: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.end,
                children: <Widget>[
                  Text(conversation.formattedLastMessageTime, style: TextStyle(color: hasUnread ? Theme.of(context).colorScheme.secondary : Colors.grey[600], fontSize: 12, fontWeight: hasUnread ? FontWeight.bold : FontWeight.normal)),
                  const SizedBox(height: 5),
                  if (hasUnread)
                    Container(
                      width: 22,
                      height: 22,
                      decoration: BoxDecoration(color: Theme.of(context).colorScheme.secondary, shape: BoxShape.circle),
                      child: Center(child: Text(conversation.unreadCount.toString(), style: const TextStyle(color: Colors.white, fontSize: 10, fontWeight: FontWeight.bold))),
                    )
                  else
                    const SizedBox(width: 22, height: 22)
                ]),
            onTap: () {
              Navigator.push(
                  context,
                  MaterialPageRoute(
                      builder: (context) => IndividualChatScreen(
                          chatName: conversation.contactName,
                          avatarLetter: conversation.avatarIdentifier)));
            },
          );
        });
  }
}

class StatusTabScreen extends StatefulWidget {
  const StatusTabScreen({super.key});
  @override
  State<StatusTabScreen> createState() => StatusTabScreenState();
}

class StatusTabScreenState extends State<StatusTabScreen> {
  late List<UserStatus> dummyRecentStatus;
  late List<UserStatus> dummyViewedStatus;

  @override
  void initState() {
    super.initState();
    dummyRecentStatus = [
      UserStatus(
        userId: 'user1',
        userName: 'صديق ١',
        userAvatarUrl: 'https://via.placeholder.com/150/007ACC/FFFFFF?Text=S1',
        lastUpdated: DateTime.now().subtract(const Duration(hours: 1)),
        stories: [
          Story(id: 's1_1', type: StoryType.image, contentUrl: 'https://picsum.photos/seed/s1_1_blue/720/1280', timestamp: DateTime.now().subtract(const Duration(hours: 1, minutes: 5))),
          Story(id: 's1_2', type: StoryType.text, textContent: "يوم رائع!", backgroundColor: MyApp.talkgramAccentBlue.withOpacity(0.7), contentUrl: 's1_2_text_placeholder', timestamp: DateTime.now().subtract(const Duration(hours: 1, minutes: 2))),
          Story(id: 's1_3', type: StoryType.image, contentUrl: 'https://picsum.photos/seed/s1_3_blue/720/1280', timestamp: DateTime.now().subtract(const Duration(hours: 1))),
        ],
      ),
      UserStatus(
        userId: 'user2',
        userName: 'مجموعة العمل',
        userAvatarUrl: 'https://via.placeholder.com/150/FF6F00/FFFFFF?Text=WG',
        lastUpdated: DateTime.now().subtract(const Duration(hours: 3)),
        stories: [ Story(id: 's2_1', type: StoryType.image, contentUrl: 'https://picsum.photos/seed/s2_1_orange/720/1280', timestamp: DateTime.now().subtract(const Duration(hours: 3))) ],
        allStoriesViewed: false,
      ),
    ];

    dummyViewedStatus = [
      UserStatus(
        userId: 'user4',
        userName: 'زميل قديم',
        userAvatarUrl: 'https://via.placeholder.com/150/4CAF50/FFFFFF?Text=ZF',
        lastUpdated: DateTime.now().subtract(const Duration(days: 2)),
        stories: [ Story(id: 's4_1', type: StoryType.image, contentUrl: 'https://picsum.photos/seed/s4_1_green/720/1280', timestamp: DateTime.now().subtract(const Duration(days: 2))) ],
        allStoriesViewed: true,
      ),
    ];
  }

  void _markStatusAsViewed(String userId) {
    if (mounted) {
      setState(() {
        final statusIndex = dummyRecentStatus.indexWhere((s) => s.userId == userId);
        if (statusIndex != -1) {
          dummyRecentStatus[statusIndex].allStoriesViewed = true;
        }
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    final recentUpdates = dummyRecentStatus.where((status) => !status.allStoriesViewed).toList();
    final viewedUpdates = dummyRecentStatus.where((status) => status.allStoriesViewed).toList();
    viewedUpdates.addAll(dummyViewedStatus.where((vs) => !viewedUpdates.any((vus) => vus.userId == vs.userId && vus.allStoriesViewed)));


    return Scaffold(
      body: ListView(
        children: <Widget>[
          ListTile(
            leading: Stack(alignment: Alignment.bottomRight, children: [const CircleAvatar(radius: 28, backgroundColor: Colors.grey, child: Icon(Icons.person, size: 30, color: Colors.white)), CircleAvatar(radius: 10, backgroundColor: Theme.of(context).colorScheme.secondary, child: const Icon(Icons.add_circle, size: 15, color: Colors.white))]),
            title: const Text("My status", style: TextStyle(fontWeight: FontWeight.bold)),
            subtitle: const Text("Tap to add status update"),
            onTap: () { ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم فتح محرر الحالة هنا!'))); }
          ),
          if (recentUpdates.isNotEmpty)
            Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0), child: Text("Recent updates", style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold))),
          ...recentUpdates.map((status) => ListTile(
            leading: CircleAvatar(radius: 28, backgroundColor: Theme.of(context).colorScheme.secondary, child: CircleAvatar(radius: 25, backgroundImage: NetworkImage(status.userAvatarUrl))),
            title: Text(status.userName, style: const TextStyle(fontWeight: FontWeight.bold)),
            subtitle: Text(status.lastUpdated.toLocal().toString().substring(0, 16)),
            onTap: () {
              final combinedList = dummyRecentStatus + dummyViewedStatus.where((vs) => !dummyRecentStatus.any((rs) => rs.userId == vs.userId)).toList();
              final initialIndex = combinedList.indexWhere((s) => s.userId == status.userId);
              if (initialIndex != -1) {
                 Navigator.push(context, MaterialPageRoute(builder: (context) => StoryViewScreen(userStatusList: combinedList, initialStatusIndex: initialIndex, onStatusViewed: _markStatusAsViewed)));
              }
            }
          )).toList(),
          if (viewedUpdates.isNotEmpty) ...[
            Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0), child: Text("Viewed updates", style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold))),
            ...viewedUpdates.map((status) => ListTile(
              leading: CircleAvatar(radius: 28, backgroundColor: Colors.grey.shade400, child: CircleAvatar(radius: 25, backgroundImage: NetworkImage(status.userAvatarUrl))),
              title: Text(status.userName, style: const TextStyle(fontWeight: FontWeight.bold)),
              subtitle: Text(status.lastUpdated.toLocal().toString().substring(0, 16)),
              onTap: () {
                final combinedList = dummyRecentStatus + dummyViewedStatus.where((vs) => !dummyRecentStatus.any((rs) => rs.userId == vs.userId)).toList();
                final initialIndex = combinedList.indexWhere((s) => s.userId == status.userId);
                if (initialIndex != -1) {
                  Navigator.push(context, MaterialPageRoute(builder: (context) => StoryViewScreen(userStatusList: combinedList, initialStatusIndex: initialIndex, onStatusViewed: _markStatusAsViewed)));
                }
              }
            )).toList()
          ]
        ]
      )
    );
  }
}

class CallsTabScreen extends StatefulWidget {
  const CallsTabScreen({super.key});
  @override
  State<CallsTabScreen> createState() => CallsTabScreenState();
}

class CallsTabScreenState extends State<CallsTabScreen> {
  final List<Map<String, dynamic>> callLog = const [
    {'name': 'صديق ١', 'avatarUrl': 'https://via.placeholder.com/150/007ACC/FFFFFF?Text=S1', 'time': 'Today, 11:05 AM', 'type': 'incoming', 'mediaType': 'voice'},
    {'name': 'مجموعة العمل', 'avatarUrl': 'https://via.placeholder.com/150/FF6F00/FFFFFF?Text=WG', 'time': 'Today, 9:45 AM', 'type': 'outgoing', 'mediaType': 'video'},
    {'name': 'أختي', 'avatarUrl': 'https://via.placeholder.com/150/4CAF50/FFFFFF?Text=Sis', 'time': 'Yesterday, 8:30 PM', 'type': 'missed', 'mediaType': 'voice'}
  ];

  Widget _getCallTypeIcon(String callType, String mediaType, BuildContext context) {
    IconData iconData;
    Color iconColor;
    if (callType == 'incoming') {
      iconData = Icons.call_received;
      iconColor = Colors.green.shade600;
    } else if (callType == 'outgoing') {
      iconData = Icons.call_made;
      iconColor = Theme.of(context).colorScheme.secondary;
    } else {
      iconData = Icons.call_missed;
      iconColor = Theme.of(context).colorScheme.error;
    }
    return Icon(iconData, color: iconColor, size: 18);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        children: <Widget>[
          ListTile(
            leading: CircleAvatar(backgroundColor: Theme.of(context).colorScheme.secondary, child: const Icon(Icons.link, color: Colors.white)),
            title: const Text("Create call link", style: TextStyle(fontWeight: FontWeight.bold)),
            subtitle: const Text("Share a link for your Talkgram call"),
            onTap: () => ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم إنشاء رابط مكالمة هنا!')))
          ),
          Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0), child: Text("Recent", style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold))),
          ...callLog.map((call) => ListTile(
            leading: CircleAvatar(
              radius: 28,
              backgroundColor: Colors.grey.shade300,
              backgroundImage: call['avatarUrl'] != null ? NetworkImage(call['avatarUrl']) : null,
              child: call['avatarUrl'] == null ? const Icon(Icons.person, size: 30, color: Colors.white) : null
            ),
            title: Text(call['name'], style: TextStyle(fontWeight: FontWeight.bold, color: call['type'] == 'missed' ? Theme.of(context).colorScheme.error : Theme.of(context).textTheme.bodyLarge?.color)),
            subtitle: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                _getCallTypeIcon(call['type'], call['mediaType'], context),
                const SizedBox(width: 6),
                Text(call['time'])
              ]
            ),
            trailing: IconButton(
              icon: Icon(call['mediaType'] == 'video' ? Icons.videocam : Icons.call, color: Theme.of(context).primaryColor),
              onPressed: () => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('جاري الاتصال بـ ${call['name']} (${call['mediaType']})...')))
            ),
            onTap: () => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('عرض تفاصيل مكالمة ${call['name']}')))
          )).toList()
        ]
      )
    );
  }
}

class IndividualChatScreen extends StatefulWidget {
  final String chatName;
  final String? avatarLetter;
  const IndividualChatScreen({super.key, required this.chatName, this.avatarLetter});
  @override
  State<IndividualChatScreen> createState() => IndividualChatScreenState();
}

class IndividualChatScreenState extends State<IndividualChatScreen> {
  final TextEditingController _messageController = TextEditingController();
  final ScrollController _scrollController = ScrollController();
  final List<ChatMessage> _messages = [
    ChatMessage(id: 'sample1', text: 'اسحب هذه الرسالة للرد!', senderId: 'other', timestamp: DateTime.now().subtract(const Duration(minutes: 2))),
    ChatMessage(id: 'sample2', text: 'أو اسحب رسالتي أنا!', senderId: 'me', timestamp: DateTime.now().subtract(const Duration(minutes: 1))),
  ];
  int _nextMessageIdCounter = 3;
  ChatMessage? _replyingToMessage;
  ChatMessage? _selectedMessage;
  bool _isSelectionMode = false;
  bool _isRecordingAudio = false;

  String _contactStatus = "متصل الآن";
  Timer? _statusTimer;

  @override
  void initState() {
    super.initState();
    _messageController.addListener(() {
      if (mounted) setState(() {});
    });

    _statusTimer = Timer.periodic(const Duration(seconds: 5), (timer) {
      if (!mounted) {
        timer.cancel();
        return;
      }
      if (mounted) {
          setState(() {
            final statuses = ["متصل الآن", "يكتب الآن...", "آخر ظهور اليوم الساعة ${DateTime.now().hour}:${DateTime.now().minute.toString().padLeft(2, '0')}"];
            _contactStatus = statuses[DateTime.now().second % statuses.length];
        });
      }
    });
  }

  @override
  void dispose() {
    _messageController.dispose();
    _scrollController.dispose();
    _statusTimer?.cancel();
    super.dispose();
  }

  void _scrollToBottom() {
    Future.delayed(const Duration(milliseconds: 100), () {
      if (_scrollController.hasClients) _scrollController.animateTo(_scrollController.position.maxScrollExtent, duration: const Duration(milliseconds: 300), curve: Curves.easeOut);
    });
  }
  void _startReply(ChatMessage message) { if (mounted) setState(() => _replyingToMessage = message); }
  void _cancelReply() { if (mounted) setState(() => _replyingToMessage = null); }

  void _onMessageLongPress(ChatMessage message) { if (mounted) setState(() { _isSelectionMode = true; _selectedMessage = message; _replyingToMessage = null; }); }
  void _onMessageTap(ChatMessage message) { if (_isSelectionMode) { if (mounted) setState(() { if (_selectedMessage?.id == message.id) { _selectedMessage = null; _isSelectionMode = false; } else { _selectedMessage = message; } }); } else { print("Normal tap on message: ${message.text}"); } }
  void _cancelSelectionMode() { if (mounted) setState(() { _isSelectionMode = false; _selectedMessage = null; }); }

  void _copySelectedMessage() { if (_selectedMessage != null) { print("Message copied (theoretically): ${_selectedMessage!.text}"); if (mounted) ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم نسخ الرسالة (وهمياً)!'))); _cancelSelectionMode(); } }
  void _starSelectedMessage() { if (_selectedMessage != null) { print("Message starred (theoretically): ${_selectedMessage!.id}"); if (mounted) ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم تمييز الرسالة بنجمة (وهمياً)!'))); _cancelSelectionMode(); } }
  void _deleteSelectedMessage() { if (_selectedMessage != null) { print("Message delete initiated (theoretically): ${_selectedMessage!.id}"); if (mounted) { final messageIdToDelete = _selectedMessage!.id; setState(() => _messages.removeWhere((msg) => msg.id == messageIdToDelete)); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم حذف الرسالة (وهمياً)!'))); } _cancelSelectionMode(); } }
  void _forwardSelectedMessage() { if (_selectedMessage != null) { print("Message forward initiated (theoretically): ${_selectedMessage!.id}"); if (mounted) ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم فتح شاشة اختيار جهة اتصال لإعادة التوجيه (وهمياً)!'))); _cancelSelectionMode(); } }
  void _messageInfo() { if (_selectedMessage != null) { print("Show message info (theoretically): ${_selectedMessage!.id}"); if (mounted) ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم عرض معلومات الرسالة (وهمياً)!'))); _cancelSelectionMode(); } }

  Future<void> _sendMessage() async {
    if (_messageController.text.trim().isNotEmpty) {
      final String messageText = _messageController.text.trim();
      final newMessage = ChatMessage(id: _nextMessageIdCounter.toString(), text: messageText, senderId: 'me', timestamp: DateTime.now(), status: MessageStatus.sending, repliedToMessageId: _replyingToMessage?.id, repliedToMessageText: _replyingToMessage?.text, repliedToSenderId: _replyingToMessage?.senderId );
      _nextMessageIdCounter++;
      if (mounted) setState(() { _messages.add(newMessage); _messageController.clear(); _replyingToMessage = null; });
      _scrollToBottom();
      await Future.delayed(const Duration(seconds: 1));
      if (mounted) setState(() { final idx = _messages.indexWhere((m) => m.id == newMessage.id); if (idx != -1) _messages[idx].status = MessageStatus.sent; });
    }
  }

  void _showAttachmentBottomSheet(BuildContext context) { showModalBottomSheet(context: context, shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(20.0))), builder: (BuildContext bc) { return SafeArea(child: Container(padding: const EdgeInsets.all(16.0), child: Wrap(spacing: 16.0, runSpacing: 16.0, alignment: WrapAlignment.center, children: <Widget>[ _buildAttachmentOption(context, Icons.photo_library, "المعرض", () { Navigator.pop(context); _pickImageFromGallery(); }), _buildAttachmentOption(context, Icons.camera_alt, "الكاميرا", () { Navigator.pop(context); _takePhotoWithCamera(); }), _buildAttachmentOption(context, Icons.insert_drive_file, "مستند", () { Navigator.pop(context); _attachDocument(); }), _buildAttachmentOption(context, Icons.location_pin, "الموقع", () { Navigator.pop(context); _shareLocation(); }), _buildAttachmentOption(context, Icons.person, "جهة اتصال", () { Navigator.pop(context); _shareContact(); }) ]))); }); }
  Widget _buildAttachmentOption(BuildContext context, IconData icon, String label, VoidCallback onPressed) { return InkWell(onTap: onPressed, borderRadius: BorderRadius.circular(12.0), child: Padding(padding: const EdgeInsets.all(8.0), child: Column(mainAxisSize: MainAxisSize.min, children: <Widget>[CircleAvatar(radius: 28, backgroundColor: Theme.of(context).primaryColor.withOpacity(0.1), child: Icon(icon, size: 28, color: Theme.of(context).primaryColor)), const SizedBox(height: 8), Text(label, style: const TextStyle(fontSize: 12))]))); }
  Future<void> _pickImageFromGallery() async { print("Pick image from Gallery called"); if (mounted) { _addAttachmentMessage("[صورة من المعرض مُرفقة]"); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم اختيار صورة من المعرض (وهمياً)!'))); }}
  Future<void> _takePhotoWithCamera() async { print("Take photo with Camera called"); if (mounted) { _addAttachmentMessage("[صورة من الكاميرا مُرفقة]"); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم التقاط صورة بالكاميرا (وهمياً)!'))); }}
  void _attachDocument() { print("Attach document called"); if (mounted) { _addAttachmentMessage("[مستند مُرفق]"); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم إرفاق مستند (وهمياً)!'))); }}
  void _shareLocation() { print("Share location called"); if (mounted) { _addAttachmentMessage("[تمت مشاركة الموقع]"); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تمت مشاركة الموقع (وهمياً)!'))); }}
  void _shareContact() { print("Share contact called"); if (mounted) { _addAttachmentMessage("[تمت مشاركة جهة اتصال]"); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تمت مشاركة جهة اتصال (وهمياً)!'))); }}
  void _addAttachmentMessage(String attachmentText) { final newMessage = ChatMessage(id: _nextMessageIdCounter.toString(), text: attachmentText, senderId: 'me', timestamp: DateTime.now(), status: MessageStatus.sent); _nextMessageIdCounter++; if (mounted) setState(() => _messages.add(newMessage)); _scrollToBottom(); }

  void _toggleAudioRecording() {
    if (_isRecordingAudio) {
      if(mounted) setState(() => _isRecordingAudio = false);
      _addAttachmentMessage("[رسالة صوتية مُرفقة 🎤]");
      if(mounted) ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم إيقاف التسجيل وإرسال الرسالة الصوتية (وهمياً)!')));
    } else {
      if(mounted) setState(() => _isRecordingAudio = true);
      if(mounted) ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('جاري تسجيل الصوت... حرر للإرسال، اسحب للإلغاء (وهمياً)!'),
          duration: Duration(seconds: 3),
        ),
      );
    }
  }

 Widget _buildMessageBubble(ChatMessage message) {
    final bool isMyMessage = message.isMe;
    final DismissDirection dismissDirection = isMyMessage ? DismissDirection.endToStart : DismissDirection.startToEnd;
    final bool isSelected = _isSelectionMode && _selectedMessage?.id == message.id;

    Widget bubbleContent = Container(
      padding: const EdgeInsets.symmetric(vertical: 8.0, horizontal: 12.0),
      decoration: BoxDecoration(
        color: isMyMessage
            ? (message.status == MessageStatus.sending ? Theme.of(context).colorScheme.secondary.withOpacity(0.7) : Theme.of(context).colorScheme.secondary)
            : Colors.grey.shade300,
        borderRadius: BorderRadius.only(
          topLeft: const Radius.circular(16.0),
          topRight: const Radius.circular(16.0),
          bottomLeft: isMyMessage ? const Radius.circular(16.0) : const Radius.circular(0.0),
          bottomRight: isMyMessage ? const Radius.circular(0.0) : const Radius.circular(16.0),
        ),
      ),
      constraints: BoxConstraints(maxWidth: MediaQuery.of(context).size.width * 0.75),
      child: Column(
        crossAxisAlignment: isMyMessage ? CrossAxisAlignment.end : CrossAxisAlignment.start,
        mainAxisSize: MainAxisSize.min,
        children: [
          if (message.repliedToMessageText != null && message.repliedToMessageText!.isNotEmpty)
            Container(
              padding: const EdgeInsets.all(8.0),
              margin: const EdgeInsets.only(bottom: 6.0),
              decoration: BoxDecoration(
                  color: isMyMessage ? Theme.of(context).primaryColor.withOpacity(0.1) : Colors.black.withOpacity(0.05),
                  borderRadius: BorderRadius.circular(8.0),
                  border: Border(left: BorderSide(color: isMyMessage ? Theme.of(context).colorScheme.secondary : Theme.of(context).primaryColor, width: 3))),
              child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
                Text(message.repliedToSenderId == 'me' ? "أنت" : (widget.chatName), style: TextStyle(fontWeight: FontWeight.bold, fontSize: 13, color: isMyMessage ? Theme.of(context).colorScheme.secondary : Theme.of(context).primaryColor)),
                const SizedBox(height: 2),
                Text(message.repliedToMessageText!, style: TextStyle(fontSize: 14, color: isMyMessage ? Colors.white.withOpacity(0.9) : Colors.black.withOpacity(0.7)), maxLines: 2, overflow: TextOverflow.ellipsis)
              ]),
            ),
          Text(message.text, style: TextStyle(color: isMyMessage ? Colors.white : Colors.black87, fontSize: 16)),
          const SizedBox(height: 4.0),
          Row(mainAxisSize: MainAxisSize.min, children: [
            Text(message.formattedTime, style: TextStyle(color: isMyMessage ? Colors.white70 : Colors.black54, fontSize: 10)),
            if (isMyMessage) ...[
              const SizedBox(width: 5),
              Icon(message.status == MessageStatus.sending ? Icons.watch_later_outlined : (message.status == MessageStatus.delivered || message.status == MessageStatus.read ? Icons.done_all : Icons.done), size: 14, color: message.status == MessageStatus.sending ? Colors.white70 : (message.status == MessageStatus.read ? Colors.lightBlueAccent.shade100 : Colors.white70))
            ]
          ])
        ],
      ),
    );

    final Widget bubbleWithTail = Stack(
      clipBehavior: Clip.none,
      children: [
        bubbleContent,
        if (!isMyMessage)
          Positioned(
            left: -6,
            bottom: 0,
            child: ClipPath(
              clipper: _MessageTailClipper(),
              child: Container(
                width: 12,
                height: 12,
                color: Colors.grey.shade300,
              ),
            ),
          ),
        if (isMyMessage)
          Positioned(
            right: -6,
            bottom: 0,
            child: ClipPath(
              clipper: _MessageTailClipper(isMyMessage: true),
              child: Container(
                width: 12,
                height: 12,
                color: message.status == MessageStatus.sending
                  ? Theme.of(context).colorScheme.secondary.withOpacity(0.7)
                  : Theme.of(context).colorScheme.secondary,
              ),
            ),
          ),
      ],
    );

    return InkWell(
        onTap: () => _onMessageTap(message),
        onLongPress: () => _onMessageLongPress(message),
        child: Dismissible(
            key: Key(message.id),
            direction: _isSelectionMode ? DismissDirection.none : dismissDirection,
            confirmDismiss: (direction) async {
              if (!_isSelectionMode) _startReply(message);
              return false;
            },
            background: _isSelectionMode ? null : Container(color: Theme.of(context).primaryColor.withOpacity(0.08), padding: const EdgeInsets.symmetric(horizontal: 20), alignment: isMyMessage ? Alignment.centerLeft : Alignment.centerRight, child: Icon(Icons.reply, color: Theme.of(context).primaryColor)),
            child: Container(
                color: isSelected ? Theme.of(context).primaryColor.withOpacity(0.15) : null,
                padding: const EdgeInsets.symmetric(vertical: 4.0, horizontal: 8.0),
                child: Align(
                    alignment: isMyMessage ? Alignment.centerRight : Alignment.centerLeft,
                    child: bubbleWithTail,
                    ))));
  }

  @override
  Widget build(BuildContext context) {
    final AppBar currentAppBar = _isSelectionMode ? AppBar(leading: IconButton(icon: const Icon(Icons.close), onPressed: _cancelSelectionMode), title: Text(_selectedMessage != null ? "1 selected" : "Select message"), backgroundColor: MyApp.talkgramDarkBlue, actions: <Widget>[ if (_selectedMessage != null && !_selectedMessage!.isMe) IconButton(icon: const Icon(Icons.info_outline), tooltip: "معلومات", onPressed: _messageInfo), IconButton(icon: const Icon(Icons.star_border), tooltip: "تمييز بنجمة", onPressed: _starSelectedMessage), IconButton(icon: const Icon(Icons.delete_outline), tooltip: "حذف", onPressed: _deleteSelectedMessage), IconButton(icon: const Icon(Icons.copy), tooltip: "نسخ", onPressed: _copySelectedMessage), IconButton(icon: const Icon(Icons.reply), tooltip: "رد", onPressed: () { if(_selectedMessage != null) _startReply(_selectedMessage!); _cancelSelectionMode(); }), IconButton(icon: const Icon(Icons.shortcut), tooltip: "إعادة توجيه", onPressed: _forwardSelectedMessage) ])
        : AppBar(
            leadingWidth: 25,
            title: Row(children: [
              CircleAvatar(radius: 18, backgroundColor: Colors.white.withOpacity(0.2), child: widget.avatarLetter != null ? Text(widget.avatarLetter!, style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold)) : const Icon(Icons.person, size: 22, color: Colors.white)),
              const SizedBox(width: 10),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text(widget.chatName, style: const TextStyle(fontSize: 18), overflow: TextOverflow.ellipsis),
                    if (_contactStatus.isNotEmpty)
                      Text(
                        _contactStatus,
                        style: TextStyle(fontSize: 12, color: Colors.white.withOpacity(0.8)),
                        overflow: TextOverflow.ellipsis,
                      ),
                  ],
                ),
              )
            ]),
            actions: [IconButton(icon: const Icon(Icons.videocam), onPressed: () => print("Video call pressed")), IconButton(icon: const Icon(Icons.call), onPressed: () => print("Audio call pressed")), PopupMenuButton<String>(onSelected: (value) => print("Chat Menu: $value"), itemBuilder: (BuildContext c) => {'View contact', 'Media, links, and docs', 'Search', 'Mute notifications', 'Disappearing messages', 'Wallpaper', 'More'}.map((String choice) => PopupMenuItem<String>(value: choice, child: Text(choice))).toList())]);

    return Scaffold(
        appBar: currentAppBar,
        body: Column(children: [
          Expanded(child: ListView.builder(controller: _scrollController, padding: const EdgeInsets.symmetric(vertical: 8.0), itemCount: _messages.length, itemBuilder: (context, index) => _buildMessageBubble(_messages[index]))),
          if (_replyingToMessage != null && !_isSelectionMode)
            Container(
                padding: const EdgeInsets.symmetric(horizontal: 12.0, vertical: 8.0),
                margin: const EdgeInsets.only(left:8.0, right: 8.0, bottom: 2.0),
                decoration: BoxDecoration(color: Theme.of(context).scaffoldBackgroundColor, border: Border(top: BorderSide(color: Colors.grey.shade300, width: 0.5))),
                child: Row(crossAxisAlignment: CrossAxisAlignment.center, children: [
                  Container(width: 4, height: 40, color: Theme.of(context).primaryColor, margin: const EdgeInsets.only(right: 8.0)),
                  Expanded(child: Column(crossAxisAlignment: CrossAxisAlignment.start, mainAxisSize: MainAxisSize.min, children: [Text(_replyingToMessage!.isMe ? "أنت" : widget.chatName, style: TextStyle(fontWeight: FontWeight.bold, color: Theme.of(context).primaryColor, fontSize: 13)), const SizedBox(height: 2), Text(_replyingToMessage!.text, maxLines: 1, overflow: TextOverflow.ellipsis, style: TextStyle(color: Colors.black54, fontSize: 14))])),
                  IconButton(icon: const Icon(Icons.close, size: 20), onPressed: _cancelReply, padding: EdgeInsets.zero, constraints: const BoxConstraints())
                ])),
          if (!_isSelectionMode)
            Container(
                padding: const EdgeInsets.symmetric(horizontal: 8.0, vertical: 8.0),
                decoration: BoxDecoration(color: Theme.of(context).scaffoldBackgroundColor, boxShadow: _replyingToMessage == null ? [BoxShadow(offset: const Offset(0, -2), blurRadius: 5, color: Colors.grey.withOpacity(0.1))] : null, border: _replyingToMessage != null ? Border(top: BorderSide(color: Colors.grey.shade300, width: 0.5)) : null),
                child: Row(children: [
                  IconButton(icon: Icon(Icons.emoji_emotions_outlined, color: Colors.grey.shade600), onPressed: () => print("Emoji pressed")),
                  Expanded(
                      child: TextField(
                          controller: _messageController,
                          decoration: InputDecoration(
                            hintText: _isRecordingAudio ? "جاري التسجيل..." : "اكتب رسالة...",
                          ),
                          textCapitalization: TextCapitalization.sentences,
                          minLines: 1,
                          maxLines: 5,
                          onSubmitted: (value) => _sendMessage())),
                  _messageController.text.trim().isNotEmpty
                      ? IconButton(icon: Icon(Icons.send, color: Theme.of(context).colorScheme.secondary), onPressed: _sendMessage)
                      : Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(icon: Icon(Icons.attach_file, color: Colors.grey.shade600), onPressed: () => _showAttachmentBottomSheet(context)),
                             GestureDetector(
                               onLongPress: _toggleAudioRecording,
                               onLongPressUp: _isRecordingAudio ? _toggleAudioRecording : null,
                               child: IconButton(
                                 icon: Icon(
                                   _isRecordingAudio ? Icons.mic_off : Icons.camera_alt_outlined,
                                   color: _isRecordingAudio ? Theme.of(context).colorScheme.error : Colors.grey.shade600,
                                 ),
                                 onPressed: _isRecordingAudio ? _toggleAudioRecording : () {
                                    print("Camera pressed (short press)");
                                    _takePhotoWithCamera();
                                  },
                               ),
                             ),
                          ],
                        )
                ]))
        ]));
  }
}

class _MessageTailClipper extends CustomClipper<Path> {
  final bool isMyMessage;
  _MessageTailClipper({this.isMyMessage = false});

  @override
  Path getClip(Size size) {
    final path = Path();
    if (isMyMessage) {
      path.moveTo(0, size.height);
      path.lineTo(size.width, size.height);
      path.lineTo(0, size.height * 0.5);
    } else {
      path.moveTo(size.width, size.height);
      path.lineTo(0, size.height);
      path.lineTo(size.width, size.height * 0.5);
    }
    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper<Path> oldClipper) => false;
}

class EditProfileScreen extends StatefulWidget {
  final String currentName;
  final String currentStatus;
  final String? currentAvatarUrl;
  const EditProfileScreen({super.key, this.currentName = "اسم المستخدم هنا", this.currentStatus = "Hey there! I am using Talkgram.", this.currentAvatarUrl});
  @override
  State<EditProfileScreen> createState() => EditProfileScreenState();
}

class EditProfileScreenState extends State<EditProfileScreen> {
  late TextEditingController _nameController;
  late TextEditingController _statusController;
  bool _imageSelectedForUpload = false;
  bool _isLoading = false;

  @override
  void initState() { super.initState(); _nameController = TextEditingController(text: widget.currentName); _statusController = TextEditingController(text: widget.currentStatus); }
  @override
  void dispose() { _nameController.dispose(); _statusController.dispose(); super.dispose(); }
  Future<void> _pickImage() async { if (mounted) { setState(() => _imageSelectedForUpload = true); ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم اختيار صورة جديدة (وهمياً)!'))); } print("Pick new profile image function called"); }
  Future<void> _saveProfileChanges() async { if (_nameController.text.trim().isEmpty) { ScaffoldMessenger.of(context).showSnackBar( SnackBar(content: Text('الاسم لا يمكن أن يكون فارغاً'), backgroundColor: Theme.of(context).colorScheme.error)); return; } if (mounted) setState(() => _isLoading = true); try { await Future.delayed(const Duration(seconds: 1)); print('New Name: ${_nameController.text.trim()}'); print('New Status: ${_statusController.text.trim()}'); if (_imageSelectedForUpload) print('New Profile Image was selected to be uploaded.'); if (mounted) { ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم حفظ تغييرات الملف الشخصي (وهمياً)!'))); Navigator.pop(context); } } catch (e) { if (mounted) ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('خطأ في حفظ التغييرات: ${e.toString()}'), backgroundColor: Theme.of(context).colorScheme.error)); } finally { if (mounted) setState(() => _isLoading = false); } }
  Widget _buildEditableField({required BuildContext context, required TextEditingController controller, required String label, required IconData icon, int maxLines = 1}) { return Padding(padding: const EdgeInsets.symmetric(vertical: 8.0), child: Row(crossAxisAlignment: CrossAxisAlignment.center, children: [Icon(icon, color: Theme.of(context).primaryColor, size: 28), const SizedBox(width: 20), Expanded(child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [Text(label, style: TextStyle(fontSize: 12, color: Theme.of(context).primaryColor)), TextFormField(controller: controller, maxLines: maxLines, decoration: const InputDecoration(contentPadding: EdgeInsets.symmetric(vertical: 8.0), border: UnderlineInputBorder()), style: const TextStyle(fontSize: 16))]))])); }
  @override
  Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('تعديل الملف الشخصي'), actions: [_isLoading ? const Padding(padding: EdgeInsets.all(16.0), child: SizedBox(width: 20, height: 20, child: CircularProgressIndicator(color: Colors.white, strokeWidth: 2.0))) : IconButton(icon: const Icon(Icons.check), onPressed: _saveProfileChanges, tooltip: 'حفظ')]), body: SingleChildScrollView(padding: const EdgeInsets.all(16.0), child: Column(crossAxisAlignment: CrossAxisAlignment.center, children: <Widget>[const SizedBox(height: 20), Stack(alignment: Alignment.bottomRight, children: [CircleAvatar(radius: 70, backgroundColor: Colors.grey.shade300, backgroundImage: _imageSelectedForUpload ? null : (widget.currentAvatarUrl != null ? NetworkImage(widget.currentAvatarUrl!) : null), child: _imageSelectedForUpload ? Icon(Icons.image_search, size: 70, color: Theme.of(context).colorScheme.secondary) : (widget.currentAvatarUrl == null ? Icon(Icons.person, size: 70, color: Colors.grey.shade700) : null)), CircleAvatar(radius: 22, backgroundColor: Theme.of(context).colorScheme.secondary, child: IconButton(icon: const Icon(Icons.camera_alt, color: Colors.white, size: 22), onPressed: _pickImage))]), const SizedBox(height: 30), _buildEditableField(context: context, controller: _nameController, label: "الاسم", icon: Icons.person), const SizedBox(height: 20), _buildEditableField(context: context, controller: _statusController, label: "الحالة (About)", icon: Icons.info_outline, maxLines: 3), const SizedBox(height: 20), ListTile(leading: Icon(Icons.phone, color: Theme.of(context).primaryColor), title: const Text("رقم الهاتف"), subtitle: const Text("+1234567890 (وهمي)"))]))); }
}

class SettingsScreen extends StatefulWidget {
  const SettingsScreen({super.key});
  @override
  State<SettingsScreen> createState() => SettingsScreenState();
}

class SettingsScreenState extends State<SettingsScreen> {
  final String userName = "اسم المستخدم هنا";
  final String userStatus = "Hey there! I am using Talkgram.";
  final String? userAvatarUrl = null;

  Widget _buildSettingsItem(BuildContext context, IconData icon, String title, String? subtitle, VoidCallback onTap) {
    return ListTile(
      leading: Icon(icon, color: Theme.of(context).listTileTheme.iconColor ?? Theme.of(context).primaryColor.withOpacity(0.8)),
      title: Text(title),
      subtitle: subtitle != null ? Text(subtitle) : null,
      onTap: onTap,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('الإعدادات')),
      body: ListView(
        children: <Widget>[
          InkWell(
            onTap: () {
              Navigator.push(context, MaterialPageRoute(builder: (context) => EditProfileScreen(currentName: userName, currentStatus: userStatus, currentAvatarUrl: userAvatarUrl)));
            },
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 20.0),
              child: Row(
                children: [
                  CircleAvatar(
                    radius: 32,
                    backgroundColor: Colors.grey.shade300,
                    backgroundImage: userAvatarUrl != null ? NetworkImage(userAvatarUrl!) : null,
                    child: userAvatarUrl == null ? Icon(Icons.person, size: 35, color: Colors.grey.shade700) : null,
                  ),
                  const SizedBox(width: 16.0),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(userName, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                        const SizedBox(height: 4.0),
                        Text(userStatus, style: TextStyle(fontSize: 14, color: Colors.grey.shade600), maxLines: 1, overflow: TextOverflow.ellipsis),
                      ],
                    ),
                  ),
                  IconButton(icon: Icon(Icons.qr_code, color: Theme.of(context).primaryColor), onPressed: () => ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم عرض QR Code هنا!')))),
                ],
              ),
            ),
          ),
          const Divider(),
          _buildSettingsItem(context, Icons.key_outlined, "الحساب", "الخصوصية، الأمان، تغيير الرقم", () => Navigator.push(context, MaterialPageRoute(builder: (context) => const AccountSettingsScreen()))),
          _buildSettingsItem(context, Icons.chat_bubble_outline, "الدردشات", "الخلفية، النسخ الاحتياطي، سجل الدردشات", () => Navigator.push(context, MaterialPageRoute(builder: (context) => const ChatSettingsScreen()))),
          _buildSettingsItem(context, Icons.notifications_outlined, "الإشعارات", "نغمات المحادثات، المجموعات، والمكالمات", () => Navigator.push(context, MaterialPageRoute(builder: (context) => const NotificationSettingsScreen()))),
          _buildSettingsItem(context, Icons.data_usage, "التخزين والبيانات", "استخدام الشبكة، التنزيل التلقائي", () => Navigator.push(context, MaterialPageRoute(builder: (context) => const DataAndStorageUsageScreen()))),
          _buildSettingsItem(context, Icons.help_outline, "المساعدة", "مركز المساعدة، اتصل بنا، سياسة الخصوصية", () => Navigator.push(context, MaterialPageRoute(builder: (context) => const HelpScreen()))),
          const Divider(indent: 70, endIndent: 16),
          _buildSettingsItem(context, Icons.group_add_outlined, "دعوة صديق", null, () => ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم فتح قائمة المشاركة لدعوة صديق!')))),
          const SizedBox(height: 20),
          Center(
            child: Column(
              children: [
                Text('from', style: TextStyle(color: Colors.grey.shade600, fontSize: 12)),
                Text('Talkgram Devs', style: TextStyle(color: Theme.of(context).primaryColor.withOpacity(0.8), fontSize: 16, fontWeight: FontWeight.w500)),
              ],
            ),
          ),
          const SizedBox(height: 20),
        ],
      ),
    );
  }
}

class AccountSettingsScreen extends StatefulWidget {
 const AccountSettingsScreen({super.key});
 @override
 State<AccountSettingsScreen> createState() => AccountSettingsScreenState();
}

class AccountSettingsScreenState extends State<AccountSettingsScreen> {
 @override Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('إعدادات الحساب')), body: ListView(children: <Widget>[ListTile(leading: Icon(Icons.privacy_tip_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('الخصوصية'), onTap: () => print('Privacy tapped')), ListTile(leading: Icon(Icons.security_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('الأمان'), onTap: () => print('Security tapped')), ListTile(leading: Icon(Icons.phonelink_setup_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('تغيير الرقم'), onTap: () => print('Change number tapped')), ListTile(leading: Icon(Icons.description_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('طلب معلومات الحساب'), onTap: () => print('Request account info tapped')), ListTile(leading: Icon(Icons.delete_outline, color: Theme.of(context).colorScheme.error), title: Text('حذف حسابي', style: TextStyle(color: Theme.of(context).colorScheme.error)), onTap: () => print('Delete my account tapped'))]));}
}

class ChatSettingsScreen extends StatefulWidget {
 const ChatSettingsScreen({super.key});
 @override
 State<ChatSettingsScreen> createState() => ChatSettingsScreenState();
}
class ChatSettingsScreenState extends State<ChatSettingsScreen> {
 Widget _buildSectionHeader(BuildContext context, String title) { return Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 12.0), child: Text(title.toUpperCase(), style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold, fontSize: 13))); }
 @override Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('إعدادات الدردشات')), body: ListView(children: <Widget>[_buildSectionHeader(context, 'عرض'), SwitchListTile(title: const Text('Enter لإرسال الرسالة'), subtitle: const Text('مفتاح Enter سيرسل رسالتك'), value: false, onChanged: (bool value) => print('Enter is send toggled: $value'), secondary: Icon(Icons.keyboard_return, color: Theme.of(context).listTileTheme.iconColor)), SwitchListTile(title: const Text('ظهور الوسائط في المعرض'), subtitle: const Text('عرض الوسائط التي تم تنزيلها حديثًا في معرض هاتفك'), value: true, onChanged: (bool value) => print('Media visibility toggled: $value'), secondary: Icon(Icons.image_outlined, color: Theme.of(context).listTileTheme.iconColor)), ListTile(leading: Icon(Icons.font_download_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('حجم الخط'), subtitle: const Text('متوسط'), onTap: () => print('Font size tapped')), const Divider(), _buildSectionHeader(context, 'أرشيف الدردشات'), SwitchListTile(title: const Text('إبقاء الدردشات مؤرشفة'), subtitle: const Text('ستبقى الدردشات المؤرشفة في الأرشيف عند تلقي رسالة جديدة'), value: true, onChanged: (bool value) => print('Keep chats archived toggled: $value'), secondary: Icon(Icons.inventory_2_outlined, color: Theme.of(context).listTileTheme.iconColor)), const Divider(), ListTile(leading: Icon(Icons.wallpaper_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('خلفية الشاشة'), onTap: () => print('Wallpaper tapped')), ListTile(leading: Icon(Icons.backup_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('النسخ الاحتياطي للدردشات'), onTap: () => print('Chat backup tapped')), ListTile(leading: Icon(Icons.history_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('سجل الدردشات'), onTap: () => print('Chat history tapped'))]));}
}

class NotificationSettingsScreen extends StatefulWidget {
 const NotificationSettingsScreen({super.key});
 @override
 State<NotificationSettingsScreen> createState() => NotificationSettingsScreenState();
}

class NotificationSettingsScreenState extends State<NotificationSettingsScreen> {
 Widget _buildSectionHeader(BuildContext context, String title) { return Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 12.0), child: Text(title.toUpperCase(), style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold, fontSize: 13))); }
 @override Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('إعدادات الإشعارات')), body: ListView(children: <Widget>[SwitchListTile(title: const Text('نغمات المحادثات'), subtitle: const Text('تشغيل الأصوات للرسائل الواردة والصادرة.'), value: true, onChanged: (bool value) => print('Conversation tones toggled: $value'), secondary: Icon(Icons.music_note_outlined, color: Theme.of(context).listTileTheme.iconColor)), const Divider(), _buildSectionHeader(context, 'الرسائل'), ListTile(title: const Text('نغمة الإشعار'), subtitle: const Text('النغمة الافتراضية (Whistle)'), onTap: () => print('Message notification tone tapped')), ListTile(title: const Text('الاهتزاز'), subtitle: const Text('افتراضي'), onTap: () => print('Message vibration tapped')), ListTile(title: const Text('إشعار منبثق'), subtitle: const Text('غير متاح'), onTap: () => print('Message popup notification tapped')), ListTile(title: const Text('الضوء'), subtitle: const Text('أبيض'), onTap: () => print('Message light tapped')), SwitchListTile(title: const Text('استخدام إشعارات ذات أولوية عالية'), subtitle: const Text('إظهار معاينات الإشعارات في أعلى الشاشة.'), value: true, onChanged: (bool value) => print('High priority message notifications toggled: $value')), const Divider(), _buildSectionHeader(context, 'المجموعات'), ListTile(title: const Text('نغمة الإشعار'), subtitle: const Text('النغمة الافتراضية (Whistle)'), onTap: () => print('Group notification tone tapped')), ListTile(title: const Text('الاهتزاز'), subtitle: const Text('افتراضي'), onTap: () => print('Group vibration tapped')), ListTile(title: const Text('الضوء'), subtitle: const Text('أبيض'), onTap: () => print('Group light tapped')), SwitchListTile(title: const Text('استخدام إشعارات ذات أولوية عالية'), subtitle: const Text('إظهار معاينات الإشعارات في أعلى الشاشة.'), value: true, onChanged: (bool value) => print('High priority group notifications toggled: $value')), const Divider(), _buildSectionHeader(context, 'المكالمات'), ListTile(title: const Text('نغمة الرنين'), subtitle: const Text('النغمة الافتراضية (Ring)'), onTap: () => print('Call ringtone tapped')), ListTile(title: const Text('الاهتزاز'), subtitle: const Text('افتراضي'), onTap: () => print('Call vibration tapped'))]));}
}

class DataAndStorageUsageScreen extends StatefulWidget {
  const DataAndStorageUsageScreen({super.key});
  @override
  State<DataAndStorageUsageScreen> createState() => DataAndStorageUsageScreenState();
}
class DataAndStorageUsageScreenState extends State<DataAndStorageUsageScreen> {
 Widget _buildSectionHeader(BuildContext context, String title) { return Padding(padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 12.0), child: Text(title.toUpperCase(), style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold, fontSize: 13))); } String _getMediaTypeLabel(String connectionType) { if (connectionType == 'wifi') return 'كل الوسائط'; return 'لا توجد وسائط'; } String _getConnectionTypeLabel(String connectionType) { if (connectionType == 'mobile_data') return 'بيانات المحمول'; if (connectionType == 'wifi') return 'Wi-Fi'; if (connectionType == 'roaming') return 'التجوال'; return ''; } void _showMediaDownloadOptions(BuildContext context, String connectionType) { showDialog(context: context, builder: (BuildContext context) { return AlertDialog(title: Text('تنزيل تلقائي (${_getConnectionTypeLabel(connectionType)})'), content: Column(mainAxisSize: MainAxisSize.min, children: <Widget>[CheckboxListTile(value: connectionType == 'wifi', onChanged: (v){}, title: const Text('الصور')), CheckboxListTile(value: false, onChanged: (v){}, title: const Text('مقاطع الصوت')), CheckboxListTile(value: connectionType == 'wifi', onChanged: (v){}, title: const Text('مقاطع الفيديو')), CheckboxListTile(value: false, onChanged: (v){}, title: const Text('المستندات'))]), actions: <Widget>[TextButton(child: const Text('إلغاء'), onPressed: () => Navigator.of(context).pop()), TextButton(child: Text('موافق', style: TextStyle(color: Theme.of(context).primaryColor)), onPressed: () => Navigator.of(context).pop())]); }); }
 @override Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('التخزين والبيانات')), body: ListView(children: <Widget>[ListTile(leading: Icon(Icons.data_saver_on_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('إدارة التخزين'), subtitle: const Text('0 بايت'), onTap: () => print('Manage storage tapped')), const Divider(), _buildSectionHeader(context, 'استخدام الشبكة'), ListTile(leading: Icon(Icons.network_check_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('استخدام الشبكة'), subtitle: const Text('0 بايت مرسلة • 0 بايت مستلمة'), onTap: () => print('Network usage tapped')), SwitchListTile(title: const Text('تقليل حجم البيانات المستخدم في المكالمات'), value: false, onChanged: (bool value) => print('Low data usage for calls toggled: $value')), const Divider(), _buildSectionHeader(context, 'التنزيل التلقائي للوسائط'), ListTile(title: const Text('أثناء استخدام بيانات المحمول'), subtitle: Text(_getMediaTypeLabel('mobile_data')), onTap: () => _showMediaDownloadOptions(context, 'mobile_data')), ListTile(title: const Text('أثناء الاتصال بشبكة Wi-Fi'), subtitle: Text(_getMediaTypeLabel('wifi')), onTap: () => _showMediaDownloadOptions(context, 'wifi')), ListTile(title: const Text('أثناء التجوال'), subtitle: Text(_getMediaTypeLabel('roaming')), onTap: () => _showMediaDownloadOptions(context, 'roaming')), const Divider(), _buildSectionHeader(context, 'إعدادات الخادم الوكيل (Proxy)'), ListTile(leading: Icon(Icons.settings_ethernet_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('إعدادات الخادم الوكيل'), subtitle: const Text('إيقاف'), onTap: () => print('Proxy settings tapped'))]));}
}

class HelpScreen extends StatefulWidget {
  const HelpScreen({super.key});
  @override
  State<HelpScreen> createState() => HelpScreenState();
}

class HelpScreenState extends State<HelpScreen> {
 @override Widget build(BuildContext context) { return Scaffold(appBar: AppBar(title: const Text('المساعدة')), body: ListView(children: <Widget>[ListTile(leading: Icon(Icons.help_center_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('مركز المساعدة'), onTap: () => print('Help Center tapped')), ListTile(leading: Icon(Icons.contact_support_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('اتصل بنا'), subtitle: const Text('أسئلة؟ تحتاج مساعدة؟'), onTap: () => print('Contact Us tapped')), ListTile(leading: Icon(Icons.policy_outlined, color: Theme.of(context).listTileTheme.iconColor), title: const Text('الشروط وسياسة الخصوصية'), onTap: () => print('Terms and Privacy Policy tapped')), ListTile(leading: Icon(Icons.info_outline, color: Theme.of(context).listTileTheme.iconColor), title: const Text('معلومات التطبيق'), onTap: () => print('App Info tapped'))]));}
}

class StoryViewScreen extends StatefulWidget {
  final List<UserStatus> userStatusList;
  final int initialStatusIndex;
  final Function(String userId)? onStatusViewed;

  const StoryViewScreen({
    super.key,
    required this.userStatusList,
    required this.initialStatusIndex,
    this.onStatusViewed,
  });

  @override
  State<StoryViewScreen> createState() => StoryViewScreenState();
}

class StoryViewScreenState extends State<StoryViewScreen> with TickerProviderStateMixin {
  late PageController _pageController;
  late List<List<AnimationController>> _animationControllersMap;
  late List<List<Animation<double>>> _animationsMap;
  Timer? _storyTimer;
  int _currentUserStatusIndex = 0;
  int _currentStoryIndexInStatus = 0;
  bool _isPaused = false;

  @override
  void initState() {
    super.initState();
    _currentUserStatusIndex = widget.initialStatusIndex;
    _pageController = PageController(initialPage: _currentUserStatusIndex);

    _animationControllersMap = List.generate(widget.userStatusList.length, (userIndex) {
      final user = widget.userStatusList[userIndex];
      return List.generate(
        user.stories.length,
        (storyIndex) => AnimationController(
          vsync: this,
          duration: user.stories[storyIndex].duration,
        ),
      );
    });

    _animationsMap = List.generate(widget.userStatusList.length, (userIndex) {
      return _animationControllersMap[userIndex]
          .map((controller) => Tween(begin: 0.0, end: 1.0).animate(controller))
          .toList();
    });
    
    WidgetsBinding.instance.addPostFrameCallback((_) {
       if (mounted) _startStoryPlayback();
    });
  }

  void _startStoryPlayback() {
    if (!mounted) return;
    _isPaused = false;
     if (_animationControllersMap.isNotEmpty && 
        _currentUserStatusIndex < _animationControllersMap.length && 
        _animationControllersMap[_currentUserStatusIndex].isNotEmpty &&
        _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length) {
      for (var controller in _animationControllersMap[_currentUserStatusIndex]) {
        if (mounted) controller.stop();
        if (mounted) controller.reset();
      }
    }

    if (_currentUserStatusIndex < widget.userStatusList.length &&
        widget.userStatusList[_currentUserStatusIndex].stories.isNotEmpty &&
        _currentStoryIndexInStatus < widget.userStatusList[_currentUserStatusIndex].stories.length) {
      
      if (mounted && _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length) {
          _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].forward();
      }
      _storyTimer?.cancel();
      _storyTimer = Timer.periodic(const Duration(milliseconds: 50), (timer) {
        if (!mounted || _isPaused) return;
         if (_currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length &&
            _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].status == AnimationStatus.completed) {
          _nextStory();
        }
      });
      final currentUserId = widget.userStatusList[_currentUserStatusIndex].userId;
      widget.onStatusViewed?.call(currentUserId);

    } else if (_currentUserStatusIndex < widget.userStatusList.length && widget.userStatusList[_currentUserStatusIndex].stories.isEmpty) {
      _nextUserStatus();
    }
  }

  void _nextStory() {
    if (!mounted) return;
    _storyTimer?.cancel();
     if (_animationControllersMap.isNotEmpty && 
        _currentUserStatusIndex < _animationControllersMap.length && 
        _animationControllersMap[_currentUserStatusIndex].isNotEmpty && 
        _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length) {
        if (mounted) _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].stop();
        if (mounted) _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].reset();
     }

    if (_currentStoryIndexInStatus < widget.userStatusList[_currentUserStatusIndex].stories.length - 1) {
      if (mounted) {
        setState(() => _currentStoryIndexInStatus++);
      }
      _startStoryPlayback();
    } else {
      _nextUserStatus();
    }
  }

  void _previousStory() {
    if (!mounted) return;
    _storyTimer?.cancel();
    if (_animationControllersMap.isNotEmpty && 
        _currentUserStatusIndex < _animationControllersMap.length && 
        _animationControllersMap[_currentUserStatusIndex].isNotEmpty && 
        _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length) {
        if (mounted) _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].stop();
        if (mounted) _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].reset();
     }

    if (_currentStoryIndexInStatus > 0) {
      if (mounted) {
        setState(() => _currentStoryIndexInStatus--);
      }
      _startStoryPlayback();
    } else {
      _previousUserStatus();
    }
  }

  void _nextUserStatus() {
    if (!mounted) return;
    _storyTimer?.cancel();
    if (_currentUserStatusIndex < widget.userStatusList.length - 1) {
      _pageController.nextPage(duration: const Duration(milliseconds: 300), curve: Curves.easeIn);
    } else {
      if (mounted) Navigator.pop(context);
    }
  }

  void _previousUserStatus() {
    if (!mounted) return;
    _storyTimer?.cancel();
    if (_currentUserStatusIndex > 0) {
      _pageController.previousPage(duration: const Duration(milliseconds: 300), curve: Curves.easeIn);
    } else {
      if (mounted) Navigator.pop(context);
    }
  }
  
  void _onPageChanged(int index) {
    if (!mounted) return;
    _storyTimer?.cancel();
    if (_animationControllersMap.isNotEmpty && 
        _currentUserStatusIndex >= 0 && // Add boundary check
        _currentUserStatusIndex < _animationControllersMap.length && 
        _animationControllersMap[_currentUserStatusIndex].isNotEmpty) {
      for (var controller in _animationControllersMap[_currentUserStatusIndex]) {
        if (mounted) controller.stop();
        if (mounted) controller.reset();
      }
    }

    if (mounted) {
      setState(() {
        _currentUserStatusIndex = index;
        _currentStoryIndexInStatus = 0;
      });
    }
    _startStoryPlayback();
  }

  @override
  void dispose() {
    _storyTimer?.cancel();
    for (var controllersList in _animationControllersMap) {
      for (var controller in controllersList) {
        controller.dispose();
      }
    }
    _pageController.dispose();
    super.dispose();
  }

 @override
  Widget build(BuildContext context) {
    if (widget.userStatusList.isEmpty) {
      return const Scaffold(backgroundColor: Colors.black, body: Center(child: Text("لا توجد حالات لعرضها.", style: TextStyle(color: Colors.white))));
    }
    
    _currentUserStatusIndex = _currentUserStatusIndex.clamp(0, widget.userStatusList.length - 1);
    final UserStatus currentUserStatusData = widget.userStatusList[_currentUserStatusIndex];
    _currentStoryIndexInStatus = _currentStoryIndexInStatus.clamp(0, currentUserStatusData.stories.isNotEmpty ? currentUserStatusData.stories.length -1 : 0);


    final Story? currentStoryToDisplay = currentUserStatusData.stories.isNotEmpty && _currentStoryIndexInStatus < currentUserStatusData.stories.length
        ? currentUserStatusData.stories[_currentStoryIndexInStatus]
        : null;

    return Scaffold(
      backgroundColor: Colors.black,
      body: GestureDetector(
        onTapDown: (details) {
          if (!mounted) return;
          final screenWidth = MediaQuery.of(context).size.width;
          if (details.globalPosition.dx < screenWidth / 3) {
            _previousStory();
          } else if (details.globalPosition.dx > screenWidth * 2 / 3) {
            _nextStory();
          }
        },
        onLongPressStart: (_) {
          if (!mounted) return;
          if (mounted) setState(() => _isPaused = true);
          _storyTimer?.cancel();
          if (_animationControllersMap.isNotEmpty &&
              _currentUserStatusIndex < _animationControllersMap.length &&
              _animationControllersMap[_currentUserStatusIndex].isNotEmpty &&
              _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length &&
              _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].isAnimating) {
            _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].stop();
          }
        },
        onLongPressEnd: (_) {
          if (!mounted) return;
          if (mounted) setState(() => _isPaused = false);
           if (_animationControllersMap.isNotEmpty && 
              _currentUserStatusIndex < _animationControllersMap.length && 
              _animationControllersMap[_currentUserStatusIndex].isNotEmpty &&
              _currentStoryIndexInStatus < _animationControllersMap[_currentUserStatusIndex].length &&
              _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].status != AnimationStatus.completed) {
            if (mounted) _animationControllersMap[_currentUserStatusIndex][_currentStoryIndexInStatus].forward();
          }
          _startStoryPlayback();
        },
        child: Stack(
          children: [
            PageView.builder(
              controller: _pageController,
              itemCount: widget.userStatusList.length,
              onPageChanged: _onPageChanged,
              itemBuilder: (context, userIndex) {
                if (userIndex >= widget.userStatusList.length) return Container(color: Colors.black);
                
                final UserStatus pageUserStatusData = widget.userStatusList[userIndex];
                
                if (userIndex == _currentUserStatusIndex && currentStoryToDisplay != null) {
                    if (currentStoryToDisplay.type == StoryType.image) {
                        return Center(
                        child: Image.network(
                            currentStoryToDisplay.contentUrl,
                            fit: BoxFit.contain,
                            loadingBuilder: (ctx, child, progress) => progress == null ? child : const Center(child: CircularProgressIndicator(color: Colors.white)),
                            errorBuilder: (ctx, err, st) => const Center(child: Icon(Icons.error_outline, color: Colors.white, size: 50)),
                        ),
                        );
                    } else {
                        return Container(
                        color: currentStoryToDisplay.backgroundColor ?? Colors.grey.shade800,
                        child: Center(
                            child: Padding(
                            padding: const EdgeInsets.all(24.0),
                            child: Text(
                                currentStoryToDisplay.textContent ?? "",
                                textAlign: TextAlign.center,
                                style: const TextStyle(color: Colors.white, fontSize: 28, fontWeight: FontWeight.bold, shadows: [Shadow(blurRadius: 10, color: Colors.black54)]),
                            ),
                            ),
                        ),
                        );
                    }
                }
                return Container(color: Colors.black);
              },
            ),

            Positioned(
              top: MediaQuery.of(context).padding.top + 10,
              left: 10.0,
              right: 10.0,
              child: Column(
                children: [
                  Row(
                    children: List.generate(currentUserStatusData.stories.length, (index) {
                      return Expanded(
                        child: Padding(
                          padding: const EdgeInsets.symmetric(horizontal: 1.5),
                          child: AnimatedBuilder(
                            animation: _animationsMap.isNotEmpty && 
                                         _currentUserStatusIndex < _animationsMap.length && 
                                         _animationsMap[_currentUserStatusIndex].isNotEmpty && // Add this check
                                         index < _animationsMap[_currentUserStatusIndex].length 
                                         ? _animationsMap[_currentUserStatusIndex][index] 
                                         : const AlwaysStoppedAnimation(0.0),
                            builder: (context, child) {
                              double value = 0.0;
                              if (index < _currentStoryIndexInStatus) {
                                value = 1.0;
                              } else if (index == _currentStoryIndexInStatus && 
                                         _animationsMap.isNotEmpty &&
                                         _currentUserStatusIndex < _animationsMap.length &&
                                          _animationsMap[_currentUserStatusIndex].isNotEmpty && // Add this check
                                         index < _animationsMap[_currentUserStatusIndex].length) {
                                value = _animationsMap[_currentUserStatusIndex][index].value;
                              }
                              return LinearProgressIndicator(
                                value: value,
                                backgroundColor: Colors.white.withOpacity(0.4),
                                valueColor: const AlwaysStoppedAnimation<Color>(Colors.white),
                                minHeight: 2.5,
                              );
                            },
                          ),
                        ),
                      );
                    }),
                  ),
                  const SizedBox(height: 10),
                  Row(
                    children: [
                      CircleAvatar(radius: 16, backgroundImage: NetworkImage(currentUserStatusData.userAvatarUrl)),
                      const SizedBox(width: 10),
                      Text(currentUserStatusData.userName, style: const TextStyle(color: Colors.white, fontSize: 15, fontWeight: FontWeight.w600, shadows: [Shadow(blurRadius: 1, color: Colors.black87)])),
                      const Spacer(),
                      IconButton(icon: const Icon(Icons.close, color: Colors.white, size: 26), onPressed: () { if(mounted) Navigator.pop(context); }),
                    ],
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class SelectContactScreen extends StatefulWidget {
  const SelectContactScreen({super.key, this.title = "Select contact"});
  final String title;

  @override
  State<SelectContactScreen> createState() => SelectContactScreenState();
}

class SelectContactScreenState extends State<SelectContactScreen> {
  final TextEditingController _searchController = TextEditingController();
  List<Map<String, String>> _allContacts = [];
  List<Map<String, String>> _filteredContacts = [];
  bool _isSearchingAppBar = false;

  @override
  void initState() {
    super.initState();
    _allContacts = List.generate(20, (index) {
      return {
        'id': 'contact_$index',
        'name': 'جهة اتصال ${index + 1}',
        'status': (index % 3 == 0) ? 'متوفر للدردشة 😎' : 'Hey there! I am using Talkgram.',
        'avatarLetter': 'ج${index + 1}',
      };
    });
    _filteredContacts = _allContacts;
    _searchController.addListener(_filterContacts);
  }

  @override
  void dispose() {
    _searchController.removeListener(_filterContacts);
    _searchController.dispose();
    super.dispose();
  }

  void _filterContacts() {
    final query = _searchController.text.toLowerCase();
    if (mounted) {
        setState(() {
        if (query.isEmpty) {
            _filteredContacts = _allContacts;
        } else {
            _filteredContacts = _allContacts
                .where((contact) => contact['name']!.toLowerCase().contains(query))
                .toList();
        }
        });
    }
  }

  AppBar _buildSelectContactAppBar() {
    if (_isSearchingAppBar) {
      return AppBar(
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios_new),
          onPressed: () {
            if (mounted) {
                setState(() {
                _isSearchingAppBar = false;
                _searchController.clear();
                });
            }
          },
        ),
        title: TextField(
          controller: _searchController,
          autofocus: true,
          decoration: const InputDecoration(
            hintText: 'Search contacts...',
            hintStyle: TextStyle(color: Colors.white70),
            border: InputBorder.none,
             fillColor: Colors.transparent,
             filled: false,
          ),
          style: const TextStyle(color: Colors.white, fontSize: 18),
        ),
        actions: [
          if (_searchController.text.isNotEmpty)
            IconButton(
              icon: const Icon(Icons.clear),
              onPressed: () { if (mounted) _searchController.clear(); },
            ),
        ],
      );
    } else {
      return AppBar(
        title: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(widget.title),
            Text(
              "${_filteredContacts.length} contacts",
              style: const TextStyle(fontSize: 12, color: Colors.white70),
            ),
          ],
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {
              if (mounted) {
                  setState(() => _isSearchingAppBar = true);
              }
            },
          ),
          PopupMenuButton<String>(
            onSelected: (value) => print("Contacts Menu: $value"),
            itemBuilder: (BuildContext c) =>
                {'Invite a friend', 'Contacts', 'Refresh', 'Help'}
                    .map((String choice) =>
                        PopupMenuItem<String>(value: choice, child: Text(choice)))
                    .toList(),
          ),
        ],
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildSelectContactAppBar(),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          ListTile(
            leading: CircleAvatar(
              backgroundColor: Theme.of(context).colorScheme.secondary,
              child: const Icon(Icons.group, color: Colors.white),
            ),
            title: const Text("New group", style: TextStyle(fontWeight: FontWeight.bold)),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم فتح شاشة إنشاء مجموعة جديدة!')));
            },
          ),
          ListTile(
            leading: CircleAvatar(
              backgroundColor: Theme.of(context).colorScheme.secondary,
              child: const Icon(Icons.person_add_alt_1, color: Colors.white),
            ),
            title: const Text("New contact", style: TextStyle(fontWeight: FontWeight.bold)),
            trailing: IconButton(icon: Icon(Icons.qr_code_scanner, color: Theme.of(context).primaryColor), onPressed: (){
               ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('فتح ماسح QR لإضافة جهة اتصال!')));
            }),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('سيتم فتح شاشة إضافة جهة اتصال جديدة!')));
            },
          ),
          const Divider(),
           Padding(
            padding: const EdgeInsets.only(left: 16.0, top: 12.0, bottom: 8.0, right: 16.0),
            child: Text("Contacts on Talkgram", style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold, fontSize: 13)),
          ),
          Expanded(
            child: _filteredContacts.isEmpty
                ? const Center(child: Text("No contacts found."))
                : ListView.builder(
                    itemCount: _filteredContacts.length,
                    itemBuilder: (context, index) {
                      final contact = _filteredContacts[index];
                      return ListTile(
                        leading: CircleAvatar(
                          backgroundColor: Colors.grey.shade200,
                          child: Text(
                            contact['avatarLetter']!,
                            style: TextStyle(color: Theme.of(context).primaryColor, fontWeight: FontWeight.bold),
                          ),
                        ),
                        title: Text(contact['name']!),
                        subtitle: Text(contact['status']!, maxLines: 1, overflow: TextOverflow.ellipsis),
                        onTap: () {
                          if (widget.title == "New Chat" || widget.title == "Select contact") {
                            Navigator.pushReplacement(
                              context,
                              MaterialPageRoute(
                                builder: (context) => IndividualChatScreen(
                                  chatName: contact['name']!,
                                  avatarLetter: contact['avatarLetter'],
                                ),
                              ),
                            );
                          } else if (widget.title == "New group") {
                             ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('تم اختيار ${contact['name']} للمجموعة (وهمياً)')));
                          }
                          print("Selected contact: ${contact['name']}");
                        },
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }
}
