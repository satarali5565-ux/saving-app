import 'dart:math';
import 'package:flutter/material.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:timezone/data/latest.dart' as tzdata;
import 'package:timezone/timezone.dart' as tz;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Notifier.init();
  runApp(const SavingApp());
}

String fmt(int v) => v
    .toString()
    .replaceAllMapped(RegExp(r'\B(?=(\d{3})+(?!\d))'), (m) => ',');

/// التذكيرات اليومية
class Notifier {
  static final _plugin = FlutterLocalNotificationsPlugin();

  static Future<void> init() async {
    tzdata.initializeTimeZones();
    tz.setLocalLocation(tz.getLocation('Asia/Baghdad'));
    const android = AndroidInitializationSettings('@mipmap/ic_launcher');
    await _plugin.initialize(const InitializationSettings(android: android));
    await _plugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.requestNotificationsPermission();
  }

  /// متوسط المبلغ العشوائي 3000 دينار
  static int perDay(int goal, int months) {
    final n = (goal / 3000 / (months * 30)).ceil();
    return n < 1 ? 1 : n;
  }

  static Future<void> schedule(int goal, int months) async {
    await _plugin.cancelAll();
    final n = perDay(goal, months);
    const start = 10 * 60; // 10:00 صباحاً
    const end = 21 * 60; // 09:00 مساءً
    for (var i = 0; i < n; i++) {
      final m =
          n == 1 ? 15 * 60 : start + ((end - start) * i / (n - 1)).round();
      final now = tz.TZDateTime.now(tz.local);
      var t = tz.TZDateTime(
          tz.local, now.year, now.month, now.day, m ~/ 60, m % 60);
      if (t.isBefore(now)) t = t.add(const Duration(days: 1));
      await _plugin.zonedSchedule(
        i + 1,
        'وقت الادخار 💰',
        'افتح التطبيق واسحب مبلغك العشوائي الآن',
        t,
        const NotificationDetails(
          android: AndroidNotificationDetails(
            'saving_channel',
            'تذكيرات الادخار',
            channelDescription: 'تذكيرات يومية لجمع مبالغ صغيرة',
            importance: Importance.high,
            priority: Priority.high,
          ),
        ),
        androidScheduleMode: AndroidScheduleMode.inexactAllowWhileIdle,
        uiLocalNotificationDateInterpretation:
            UILocalNotificationDateInterpretation.absoluteTime,
        matchDateTimeComponents: DateTimeComponents.time,
      );
    }
  }
}

class SavingApp extends StatelessWidget {
  const SavingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ادّخار',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.teal,
      ),
      builder: (context, child) =>
          Directionality(textDirection: TextDirection.rtl, child: child!),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  static const goal = 750000;
  int saved = 0;
  int months = 4;
  int draw = 0; // المبلغ المسحوب حالياً (0 = لا يوجد)
  SharedPreferences? _p;
  final _rnd = Random();

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    _p = await SharedPreferences.getInstance();
    setState(() {
      saved = _p!.getInt('saved') ?? 0;
      months = _p!.getInt('months') ?? 4;
      draw = _p!.getInt('draw') ?? 0;
    });
    await Notifier.schedule(goal, months);
  }

  void _persist() {
    _p?.setInt('saved', saved);
    _p?.setInt('months', months);
    _p?.setInt('draw', draw);
  }

  // مبلغ عشوائي من 1000 إلى 5000 بخطوة 500
  void _drawAmount() {
    setState(() => draw = 1000 + 500 * _rnd.nextInt(9));
    _persist();
  }

  void _confirm() {
    setState(() {
      saved += draw;
      draw = 0;
    });
    _persist();
  }

  void _skip() {
    setState(() => draw = 0);
    _persist();
  }

  Future<void> _setMonths(int m) async {
    setState(() => months = m);
    _persist();
    await Notifier.schedule(goal, m);
  }

  Future<void> _reset() async {
    final ok = await showDialog<bool>(
      context: context,
      builder: (c) => AlertDialog(
        title: const Text('البدء من جديد؟'),
        content: const Text('سيتم تصفير المبلغ المدّخر.'),
        actions: [
          TextButton(
              onPressed: () => Navigator.pop(c, false),
              child: const Text('إلغاء')),
          TextButton(
              onPressed: () => Navigator.pop(c, true),
              child: const Text('تصفير')),
        ],
      ),
    );
    if (ok == true) {
      setState(() {
        saved = 0;
        draw = 0;
      });
      _persist();
    }
  }

  @override
  Widget build(BuildContext context) {
    final remaining = max(0, goal - saved);
    final progress = (saved / goal).clamp(0.0, 1.0).toDouble();
    final done = saved >= goal;

    return Scaffold(
      appBar: AppBar(
        title: const Text('ادّخار'),
        actions: [
          IconButton(
              onPressed: _reset,
              icon: const Icon(Icons.refresh),
              tooltip: 'البدء من جديد'),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(20),
        children: [
          Text('${fmt(saved)} دينار',
              textAlign: TextAlign.center,
              style: Theme.of(context)
                  .textTheme
                  .displaySmall
                  ?.copyWith(fontWeight: FontWeight.bold)),
          Text('من هدف ${fmt(goal)} دينار',
              textAlign: TextAlign.center),
          const SizedBox(height: 16),
          ClipRRect(
            borderRadius: BorderRadius.circular(10),
            child: LinearProgressIndicator(value: progress, minHeight: 16),
          ),
          const SizedBox(height: 8),
          Text(
              done
                  ? '🎉 مبارك! بلغتَ هدفك'
                  : 'المتبقي: ${fmt(remaining)} دينار (${(progress * 100).toStringAsFixed(0)}%)',
              textAlign: TextAlign.center),
          const SizedBox(height: 28),
          Card(
            child: Padding(
              padding: const EdgeInsets.all(20),
              child: draw == 0
                  ? Column(children: [
                      const Text('جاهز لخطوة ادخار جديدة؟',
                          style: TextStyle(fontSize: 18)),
                      const SizedBox(height: 16),
                      FilledButton.icon(
                        onPressed: done ? null : _drawAmount,
                        icon: const Icon(Icons.casino),
                        label: const Text('اسحب مبلغاً عشوائياً'),
                      ),
                    ])
                  : Column(children: [
                      const Text('ضع جانباً الآن:',
                          style: TextStyle(fontSize: 18)),
                      const SizedBox(height: 8),
                      Text('${fmt(draw)} دينار',
                          style: Theme.of(context)
                              .textTheme
                              .headlineLarge
                              ?.copyWith(fontWeight: FontWeight.bold)),
                      const SizedBox(height: 16),
                      Row(children: [
                        Expanded(
                          child: FilledButton.icon(
                            onPressed: _confirm,
                            icon: const Icon(Icons.check),
                            label: const Text('تم الادخار'),
                          ),
                        ),
                        const SizedBox(width: 12),
                        OutlinedButton(
                            onPressed: _skip, child: const Text('تخطّي')),
                      ]),
                    ]),
            ),
          ),
          const SizedBox(height: 28),
          const Text('مدة الوصول للهدف',
              style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          SegmentedButton<int>(
            segments: const [
              ButtonSegment(value: 3, label: Text('3 أشهر')),
              ButtonSegment(value: 4, label: Text('4 أشهر')),
              ButtonSegment(value: 5, label: Text('5 أشهر')),
            ],
            selected: {months},
            onSelectionChanged: (s) => _setMonths(s.first),
          ),
          const SizedBox(height: 8),
          Text('عدد التذكيرات يومياً: ${Notifier.perDay(goal, months)}'),
        ],
      ),
    );
  }
}
