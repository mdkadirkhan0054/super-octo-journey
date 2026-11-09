# DEEN (দ্বীন) --- সম্পূর্ণ অ্যাপ আর্কিটেকচার ও টেকনিক্যাল স্পেসিফিকেশন

**ডকুমেন্টের ধরন:** Product Requirements + UX Specification + Application
Architecture + Database Design + GitHub JSON Content Repository\
**প্ল্যাটফর্ম:** Android-first, React Native + Expo\
**ডেটা নীতি:** Offline-first; SQLite হলো অ্যাপের local source of truth\
**UI ভাষা:** English ও Arabic בלבד --- অ্যাপের UI-তে বাংলা লেবেল/টেক্সট
থাকবে না\
**প্রধান রং:** Green\
**নেভিগেশন:** Home, Alarm, Settings --- নিচে স্থায়ী তিনটি আইকন

> এই ডকুমেন্টে যেসব বিষয় ব্যবহারকারীর সিদ্ধান্ত হিসেবে নিশ্চিত, সেগুলোকে
> **Confirmed** বলা হয়েছে। যেসব বিষয়ে সিদ্ধান্ত এখনো বাকি, সেগুলো **Open
> Decision** হিসেবে চিহ্নিত। উদাহরণ সময়গুলো কেবল mock data, বাস্তব নামাজের
> সময় নয়।

------------------------------------------------------------------------

## 1. পণ্যের উদ্দেশ্য ও মূলনীতি

DEEN একটি ব্যক্তিগত ইসলামিক দৈনন্দিন ব্যবহারের অ্যাপ। এটি কোনো social
network, public community বা account-কেন্দ্রিক অ্যাপ নয়।

### 1.1 মূলনীতি

1.  **Offline-first:** ইন্টারনেট ছাড়াই মূল ফিচার, সময়সূচি, অ্যালার্ম, সেশন
    এবং আগে থেকে ডাউনলোড করা কনটেন্ট চলবে।
2.  **SQLite source of truth:** ব্যবহারকারীর সেটিংস, সময়, অ্যালার্ম, অগ্রগতি
    ও কনটেন্টের local copy SQLite-এ থাকবে।
3.  **GitHub শুধু content distribution:** GitHub JSON repository থেকে
    যাচাইকৃত কনটেন্ট আপডেট হবে। ব্যবহারকারীর ব্যক্তিগত তথ্য GitHub-এ আপলোড হবে
    না।
4.  **Independent time systems:** Prayer Time Library-এর গণনা করা আযানের
    সময় এবং Settings-এ ব্যবহারকারীর দেওয়া জামাত/তাহাজ্জুদ সময় সম্পূর্ণ আলাদা।
5.  **No unnecessary account:** ব্যক্তিগত ব্যবহারের জন্য login/account
    বাধ্যতামূলক নয়।
6.  **Verified religious content:** হাদিস/যিকিরের আরবি, অনুবাদ, সূত্র, মান
    ও লাইসেন্সের তথ্য সংরক্ষণ করতে হবে। যাচাইহীন ধর্মীয় তথ্যকে verified
    হিসেবে দেখানো যাবে না।
7.  **No in-app theme switch:** Android System Theme অনুসরণ করবে। অ্যাপের
    ভেতরে Light/Dark toggle থাকবে না।
8.  **UI language:** সাধারণ UI English; ধর্মীয় নাম/মূল আরবি পাঠ Arabic।
    বাংলা UI text থাকবে না।

------------------------------------------------------------------------

## 2. প্রযুক্তিগত স্ট্যাক

  -----------------------------------------------------------------------
  ক্ষেত্র                   প্রস্তাবিত প্রযুক্তি         ব্যবহার
  ----------------------- ----------------------- -----------------------
  App framework           React Native + Expo     Android-first mobile
                                                  app

  Navigation              Expo Router             তিনটি bottom tab ও
                                                  nested pages

  Local database          expo-sqlite             offline data
                                                  persistence

  Notifications           expo-notifications +    alarm ও scheduled
                          প্রয়োজনমতো native        notifications
                          Android module          

  Prayer calculation      adhan                   location/date/method
                                                  অনুযায়ী prayer times

  State                   Zustand                 UI/application state

  Date/time               date-fns                date ও time utility

  Network                 expo-network            sync-এর জন্য network
                                                  state

  Haptics                 expo-haptics + প্রয়োজনে  vibration trigger
                          native Android          
                          vibration               

  Keep awake              expo-keep-awake         কেবল প্রয়োজনীয় active
                                                  session UI-তে

  Content sync            HTTPS GitHub            JSON manifest ও content
                          raw/release endpoint    files

  Validation              Zod বা সমমানের schema   JSON validation
                          validator               

  Hash                    SHA-256                 downloaded file
                                                  integrity
  -----------------------------------------------------------------------

### 2.1 Android বাস্তবতা

-   বাস্তব ডিভাইসে notification/vibration পরীক্ষা করতে Expo Go-এর বদলে
    **Expo Development Build** ব্যবহার করতে হবে।
-   Android notification permission, exact alarm permission, battery
    optimization, Do Not Disturb এবং manufacturer-specific background
    restrictions বিবেচনা করতে হবে।
-   অ্যাপ বন্ধ থাকলেও নির্ভরযোগ্য exact-time vibration দরকার হলে native
    Android implementation লাগতে পারে।
-   শুধু JavaScript timer বা সাধারণ Expo notification দিয়ে নির্দিষ্ট
    vibration duration সব Android ডিভাইসে নিশ্চিত করা যায় না।
-   OS অনুমতি না দিলে অ্যাপ সেটি bypass করবে না; Settings screen-এ
    permission/status ব্যাখ্যা করবে।
-   vibration duration-এর বাস্তব আচরণ ডিভাইসভেদে পরীক্ষা করতে হবে।

------------------------------------------------------------------------

## 3. Navigation Architecture

Bottom navigation-এ ঠিক তিনটি item থাকবে:

1.  **Home**
2.  **Alarm**
3.  **Settings**

Hadith, Dhikr, Sessions এবং History আলাদা bottom tab হবে না। এগুলো Home
বা সংশ্লিষ্ট nested navigation থেকে খোলা যাবে।

``` text
App
├── Home (Bottom Tab)
│   ├── Event Box
│   ├── Prayer/Event Cards
│   ├── Hadith
│   ├── Dhikr
│   ├── Sessions
│   └── History
├── Alarm (Bottom Tab)
│   └── Add/Edit Alarm
└── Settings (Bottom Tab)
    ├── Jamaat Times
    ├── Tahajjud Time
    ├── Prayer Calculation / Location
    └── App Permissions & Status
```

------------------------------------------------------------------------

## 4. Home Screen --- নির্দিষ্ট UI কাঠামো

Home content-এ ঠিক **১১টি fixed UI element** থাকবে:

-   ১টি উপরের rectangular **Event Box**
-   এর নিচে ১০টি wide rounded rectangular card
-   Bottom navigation এই ১১টির মধ্যে গণনা হবে না।

### 4.1 Event Box

Event Box-এ থাকবে:

-   Hijri date ও Arabic month
-   পাঁচ ওয়াক্তের calculated Adhan schedule
-   Tahajjud time
-   Sunrise
-   Sunset
-   Sehri
-   Iftar
-   প্রতিটি event-এর উপযুক্ত icon
-   English name এবং ধর্মীয় নাম/Arabic label যেখানে প্রযোজ্য
-   বর্তমান/সংশ্লিষ্ট prayer-কে highlight করা যেতে পারে

Event Box একই সঙ্গে selected card-এর detail display area। আলাদা দ্বিতীয়
detail box তৈরি করা হবে না।

### 4.2 দশটি Card

কার্ডের ক্রম:

1.  Fajr
2.  Dhuhr
3.  Asr
4.  Maghrib
5.  Isha
6.  Tahajjud
7.  Sunrise
8.  Sunset
9.  Sehri
10. Iftar

প্রতিটি কার্ডের content order:

**Icon → English name → Arabic name → Time**

### 4.3 Card interaction

-   সব card-এর আকার স্থির থাকবে।
-   Tap করলে card expand/grow করবে না।
-   Selected card-এর চারপাশে Green primary border থাকবে।
-   নির্বাচিত card-এর তথ্য উপরের Event Box-এ দেখানো হবে।
-   অন্য card tap করলে Event Box-এর content বদলাবে।
-   পাঁচ ওয়াক্তের prayer card নির্বাচন করলে Event Box-এ সংশ্লিষ্ট নামাজের
    Rak'ah/Sunnah information table দেখানো হবে।
-   Tahajjud নির্বাচন করলে Tahajjud-সংক্রান্ত তথ্য দেখানো হবে।
-   Sunrise, Sunset, Sehri, Iftar-এর জন্য Rak'ah table থাকবে না।
-   চারটি event-এর জন্য অতিরিক্ত religious description বা ruling নিজে থেকে
    যোগ করা যাবে না; verified content থাকলে তবেই দেখানো হবে।

### 4.4 Mock time data

নিচের সময়গুলো UI mock-up/test-এর জন্য; এগুলো বাস্তব prayer time নয়।

  Event        Mock time
  ---------- -----------
  Fajr          04:23 AM
  Dhuhr         01:30 PM
  Asr           04:30 PM
  Maghrib       06:10 PM
  Isha          07:30 PM
  Tahajjud      03:00 AM
  Sunrise       05:00 AM
  Sunset        06:00 PM
  Sehri         04:00 AM
  Iftar         06:00 PM

------------------------------------------------------------------------

## 5. সময় ব্যবস্থাপনা --- দুটি সম্পূর্ণ স্বাধীন সিস্টেম

এটি DEEN-এর সবচেয়ে গুরুত্বপূর্ণ business rule।

### 5.1 System A --- Prayer Time Library

এই সিস্টেম স্বয়ংক্রিয়ভাবে prayer/event time হিসাব করবে।

**৯টি event:**

-   Fajr
-   Dhuhr
-   Asr
-   Maghrib
-   Isha
-   Sunrise
-   Sunset
-   Sehri end
-   Iftar

Prayer calculation-এর input:

-   latitude/longitude
-   local date
-   timezone
-   calculation method
-   madhab/asr setting
-   high-latitude rule, যদি প্রযোজ্য হয়
-   user-selected calculation preferences

`adhan` library ব্যবহার করা যেতে পারে। Sehri end সাধারণত Fajr-এর সঙ্গে
সম্পর্কিত হলেও app-এর নির্বাচিত calculation convention ও user-facing label
স্পষ্ট করতে হবে। Iftar সাধারণত Maghrib time-এর সঙ্গে সম্পর্কিত; কোনো আলাদা
offset/rule যোগ করার আগে সেটি নির্ধারণ করতে হবে।

**Independence rule:** Library-এর calculated Adhan time বদলালে
Settings-এর Jamaat time নিজে থেকে বদলাবে না।

### 5.2 System B --- User-defined Jamaat & Tahajjud

Settings-এ ব্যবহারকারী নিজে সময় নির্ধারণ করবেন:

-   পাঁচটি Jamaat time: Fajr, Dhuhr, Asr, Maghrib, Isha
-   একটি Tahajjud time

মোট ৬টি manual event।

**Independence rule:**

-   Library Adhan time এবং manual Jamaat time আলাদা database record হবে।
-   একটির পরিবর্তনে অন্যটি update হবে না।
-   একই দিনে Adhan ও Jamaat-এর সময় কাছাকাছি বা একই হলেও দুইটি event
    identity আলাদা থাকবে।
-   Event trigger ID-ও আলাদা হবে।

### 5.3 Vibration policy

  Event group                                  Event count    Duration
  ---------------------------------------- --------------- -----------
  Settings-এর পাঁচ Jamaat time + Tahajjud                 6   3 seconds
  Prayer Time Library-এর vibration           Open Decision   নিচে দেখুন

Library vibration সম্পর্কে আলোচনায় একাধিক নির্দেশ এসেছে: পাঁচ prayer,
Tahajjud ও চারটি extra card-কে ১ মিনিট vibration দেওয়ার কথা এসেছে; আবার
চার extra card-এর vibration না থাকার কথাও আগে বলা হয়েছিল। তাই Library-এর
১ মিনিট vibration-এর চূড়ান্ত scope এখনো **Open Decision**।

Confirmed: - Settings-এর ৬টি manual event-এ ৩ সেকেন্ড vibration। - কোনো
event-এ music/ringtone ব্যবহার হবে না। - vibration pattern এখনো নির্দিষ্ট
করা হয়নি।

### 5.4 Event collision

একই সময়ে একাধিক event পড়লে:

-   প্রতিটি event-এর নিজস্ব `event_type`, `source_system` ও `trigger_key`
    থাকবে।
-   একই trigger duplicate schedule না করার জন্য unique key ব্যবহার করতে
    হবে।
-   OS notification scheduling-এর সীমাবদ্ধতা অনুযায়ী collision policy
    নির্ধারণ করতে হবে।
-   একই সময়ে একাধিক vibration হলে merge, sequential playback, নাকি
    priority---এটি **Open Decision**।
-   কোনো event অন্য event-কে নীরবে overwrite করবে না।

### 5.5 Timer architecture

Countdown বা session timer-এর জন্য প্রতি সেকেন্ডে database update করা যাবে
না।

সংরক্ষণ করতে হবে:

-   `start_at`
-   `end_at`
-   `status`

UI-তে অবশিষ্ট সময় `end_at - current_time` থেকে হিসাব হবে। অ্যাপ
background/foreground হলে বর্তমান timestamp দিয়ে UI পুনরায় হিসাব করবে।

------------------------------------------------------------------------

## 6. Alarm Screen

### 6.1 Alarm list

প্রতিটি alarm card-এ থাকবে:

-   Alarm time
-   Repeat label: Once / Daily
-   ডান পাশে On/Off toggle

নিচের ডানদিকে Floating Plus button থাকবে। এটি Add Alarm screen খুলবে।

### 6.2 Add/Edit Alarm

-   X button: cancel করে আগের screen-এ ফিরে যাবে।
-   Checkmark: validation করে save করবে এবং আগের screen-এ ফিরে যাবে।
-   কেন্দ্রে দুই-column wheel picker:
    -   Hour
    -   Minute
-   Picker-এ ছয়টি visible row থাকবে।
-   Selected value মাঝের row-তে থাকবে; অন্য row faded হবে।
-   AM/PM selector থাকবে।
-   Repeat: Once অথবা Daily।
-   Vibration toggle থাকবে।
-   Music/ringtone option থাকবে না।

### 6.3 Alarm independence

Personal Alarm, Prayer Time Library event এবং Jamaat/Tahajjud event
আলাদা feature ও database records। একটির toggle বা schedule অন্যটিকে
পরিবর্তন করবে না।

Once alarm trigger হলে সেটি completed/disabled state-এ যেতে পারে; exact
post-trigger behavior implementation-এর আগে নির্ধারণ করতে হবে।

------------------------------------------------------------------------

## 7. Settings Screen

### 7.1 Jamaat time controls

পাঁচটি horizontal prayer row/card থাকবে:

-   Fajr
-   Dhuhr
-   Asr
-   Maghrib
-   Isha

প্রতিটি row-তে:

-   Prayer name
-   Library/default Adhan time
-   ডান পাশে Edit/Plus icon
-   user-defined Jamaat time

Edit করলে Alarm-এর মতো একই Hour/Minute wheel picker খুলবে। Save করলে শুধু
manual Jamaat time update হবে।

### 7.2 Tahajjud setting

Tahajjud-এর জন্য একটি পৃথক manual time setting থাকবে। এটি ছয় নম্বর
Settings-managed event।

### 7.3 Prayer calculation settings

Prayer Time Library সঠিকভাবে কাজ করার জন্য আলাদা configuration section
থাকতে পারে:

-   Location source / coordinates
-   Calculation method
-   Asr madhab
-   Timezone
-   High-latitude rule

এগুলো Jamaat time-এর সঙ্গে মিশবে না।

### 7.4 Theme

-   Android System Theme অনুসরণ করবে।
-   In-app theme toggle থাকবে না।

------------------------------------------------------------------------

## 8. Sessions, Hadith, Dhikr ও History

### 8.1 Session concept

পাঁচ ওয়াক্ত নামাজের প্রতিটির পর ৩০ মিনিটের session পরিকল্পনা করা হয়েছে।

-   প্রতি session: ৩০ মিনিট
-   পাঁচ session পূর্ণ হলে দৈনিক মোট: ১৫০ মিনিট
-   Content type: Hadith ও Dhikr

একটি সম্ভাব্য routine আগে আলোচনায় এসেছে:

  Prayer    Proposed content
  --------- ------------------
  Fajr      Dhikr
  Dhuhr     Hadith
  Asr       Dhikr
  Maghrib   Hadith
  Isha      Dhikr

এই mapping **এখনো user-confirmed নয়**; implementation-এর আগে নিশ্চিত করতে
হবে।

### 8.2 Session behavior

-   session start/end timestamp-ভিত্তিক হবে।
-   `start_at` ও `end_at` সংরক্ষণ করতে হবে।
-   app বন্ধ হলে elapsed time timestamp থেকে পুনর্গণনা হবে।
-   session content Hadith বা Dhikr হবে।
-   Session screen-এর পূর্ণ UX এখনো নির্দিষ্ট নয়; নিজে থেকে UI flow স্থির করা
    যাবে না।

### 8.3 History

History-তে session completion/progress রাখা যেতে পারে। কোন কোন
statistic, filter ও retention দরকার তা এখনো নির্ধারিত নয়।

------------------------------------------------------------------------

## 9. Hadith & Dhikr Content Governance

### 9.1 Hadith record-এর আবশ্যিক তথ্য

প্রতিটি Hadith-এর জন্য:

-   Stable unique ID
-   Collection
-   Book/chapter
-   Hadith number
-   Arabic text
-   English translation
-   Grade/status, যদি নির্ভরযোগ্য সূত্রে থাকে
-   Source/reference
-   Edition information
-   Translator
-   Translation license/permission
-   Topics/tags
-   Verification status
-   Content revision/version

Hadith number একা global unique identifier নয়। Collection ও edition
context-সহ record শনাক্ত করতে হবে।

### 9.2 Verification status

প্রস্তাবিত status:

-   `draft`
-   `reviewed`
-   `verified`
-   `deprecated`

শুধু `verified` content-কে verified হিসেবে UI-তে দেখানো যাবে। কোনো grade
বা authenticity claim অনুমান করে যোগ করা যাবে না।

### 9.3 Translation licensing

English translation কপি/বিতরণের আগে license বা permission যাচাই করতে
হবে। Repository-তে translator ও license metadata রাখতে হবে।

### 9.4 Dhikr record

প্রস্তাবিত field:

-   Stable ID
-   Arabic text
-   English meaning/translation
-   Source/reference
-   Recommended count, যদি verified source-এ থাকে
-   Topic
-   Verification status
-   License/attribution, প্রযোজ্য হলে

ধর্মীয় count, virtue বা reward নিজে থেকে তৈরি করা যাবে না।

------------------------------------------------------------------------

## 10. GitHub JSON Content Repository

GitHub repository-এর উদ্দেশ্য হলো verified content update বিতরণ করা; app
user data সংরক্ষণ করা নয়।

### 10.1 Proposed repository structure

``` text
deen-library/
├── README.md
├── manifest.json
├── metadata/
│   ├── collections.json
│   ├── narrators.json
│   └── topics.json
├── collections/
│   ├── sahih-bukhari/
│   │   ├── collection.json
│   │   └── hadiths-001.json
│   ├── sahih-muslim/
│   │   ├── collection.json
│   │   └── hadiths-001.json
│   ├── sunan-abu-dawud/
│   ├── jami-at-tirmidhi/
│   ├── sunan-an-nasai/
│   └── sunan-ibn-majah/
└── adhkar/
    ├── morning.json
    ├── evening.json
    └── general.json
```

শুরুতে ছোট verified dataset দিয়ে শুরু করতে হবে। যাচাই ও license workflow
প্রতিষ্ঠিত হওয়ার পর ধীরে ধীরে collection বাড়াতে হবে।

### 10.2 `manifest.json` example

নিচেরটি schema/example; hash অবশ্যই বাস্তব file bytes থেকে SHA-256 দিয়ে
গণনা করতে হবে।

``` json
{
  "schema_version": 1,
  "content_version": "1.0.0",
  "generated_at": "2026-01-01T00:00:00Z",
  "files": [
    {
      "path": "metadata/collections.json",
      "sha256": "COMPUTE_REAL_SHA256",
      "size_bytes": 1200,
      "required": true
    },
    {
      "path": "collections/sahih-bukhari/hadiths-001.json",
      "sha256": "COMPUTE_REAL_SHA256",
      "size_bytes": 24000,
      "required": false
    }
  ]
}
```

`COMPUTE_REAL_SHA256` placeholder production manifest-এ রাখা যাবে না।

### 10.3 `collections.json` example

``` json
{
  "schema_version": 1,
  "collections": [
    {
      "id": "sahih-bukhari",
      "name_ar": "صحيح البخاري",
      "name_en": "Sahih al-Bukhari",
      "compiler": "Imam al-Bukhari",
      "source_reference": "Edition-specific reference required",
      "verification_status": "reviewed"
    },
    {
      "id": "sahih-muslim",
      "name_ar": "صحيح مسلم",
      "name_en": "Sahih Muslim",
      "compiler": "Imam Muslim",
      "source_reference": "Edition-specific reference required",
      "verification_status": "reviewed"
    }
  ]
}
```

### 10.4 Hadith JSON example

``` json
{
  "schema_version": 1,
  "records": [
    {
      "id": "bukhari-example-000001",
      "collection_id": "sahih-bukhari",
      "book_id": "book-example",
      "chapter_id": "chapter-example",
      "hadith_number": "1",
      "edition": {
        "name": "Edition name",
        "publisher": "Publisher",
        "year": null
      },
      "text_ar": "Verified Arabic text goes here",
      "translations": [
        {
          "language": "en",
          "text": "Verified English translation goes here",
          "translator": "Translator name",
          "license": "License or permission reference"
        }
      ],
      "grade": {
        "label": null,
        "source": null
      },
      "topics": ["intention"],
      "source_references": [
        {
          "type": "print",
          "reference": "Volume/page or edition-specific locator"
        }
      ],
      "verification_status": "draft",
      "verified_by": null,
      "verified_at": null,
      "content_revision": 1
    }
  ]
}
```

Example-এর placeholder text কোনো বাস্তব Hadith নয় এবং UI-তে দেখানো যাবে
না।

### 10.5 Dhikr JSON example

``` json
{
  "schema_version": 1,
  "records": [
    {
      "id": "dhikr-example-000001",
      "text_ar": "Verified Arabic text goes here",
      "translations": [
        {
          "language": "en",
          "text": "Verified English meaning goes here",
          "translator": "Translator name",
          "license": "License or permission reference"
        }
      ],
      "source_references": [],
      "recommended_count": null,
      "topics": [],
      "verification_status": "draft",
      "content_revision": 1
    }
  ]
}
```

### 10.6 Content sync algorithm

1.  App start বা network ফিরে এলে sync eligibility পরীক্ষা।
2.  GitHub manifest fetch।
3.  `schema_version` supported কি না যাচাই।
4.  local `sync_meta`-এর version/hash-এর সঙ্গে compare।
5.  শুধু changed file download।
6.  File size limit, JSON parse ও schema validation।
7.  SHA-256 hash compare।
8.  ID uniqueness, foreign key references, required metadata ও
    verification fields validate।
9.  Database transaction শুরু।
10. validated content upsert/import।
11. সব import সফল হলে transaction commit।
12. `sync_meta` update।
13. ব্যর্থ হলে rollback; আগের valid local content অক্ষত থাকবে।

### 10.7 Sync safety

-   Downloaded JSON-কে trusted executable code হিসেবে ব্যবহার করা যাবে
    না।
-   Remote JSON থেকে JavaScript execute করা যাবে না।
-   App user data, settings, alarm, location history বা progress
    GitHub-এ পাঠানো যাবে না।
-   Network না থাকলে local data-ই ব্যবহার হবে।
-   App প্রতিবার খুললেই পুরো library download করবে না।
-   Background sync Android-এর OS scheduling policy অনুযায়ী best-effort
    হবে।
-   Content delete/deprecate করার policy manifest ও schema-তে স্পষ্ট করতে
    হবে।

------------------------------------------------------------------------

## 11. SQLite Database Architecture

SQLite হলো local source of truth। সব user settings ও content read/write
local database-এর মাধ্যমে হবে।

### 11.1 Database conventions

-   Primary key: stable text ID বা integer ID; content records-এর ক্ষেত্রে
    stable text ID সুবিধাজনক।
-   Time: UTC epoch milliseconds অথবা ISO-8601 timestamp---একটি
    consistent convention বেছে নিতে হবে। Local wall-clock time আলাদা
    field হিসেবে রাখুন।
-   Foreign key enable করতে হবে।
-   Schema migration versioned হবে।
-   Query-তে প্রয়োজনীয় index রাখতে হবে।
-   Multi-table content import transaction-এর মধ্যে হবে।
-   Local user data remote content sync দ্বারা overwrite করা যাবে না।

### 11.2 Core tables

#### Content tables

  Table                   Purpose
  ----------------------- ---------------------------
  `collections`           Hadith collections
  `books`                 Collection-এর book
  `chapters`              Book-এর chapter
  `hadiths`               Arabic Hadith মূল record
  `hadith_translations`   একাধিক ভাষার translation
  `topics`                Shared topics/tags
  `hadith_topics`         Hadith-topic many-to-many
  `adhkar`                Dhikr content
  `adhkar_translations`   Dhikr translation
  `adhkar_topics`         Dhikr-topic many-to-many

#### User/activity tables

  Table               Purpose
  ------------------- ----------------------------------
  `sessions`          Session start/end/status
  `session_items`     Session-এর content reference
  `user_progress`     User completion/progress
  `history_entries`   প্রয়োজন হলে activity history
  `app_settings`      General app preferences
  `sync_meta`         Content sync version/hash/status

#### Time/alarm tables

  Table                 Purpose
  --------------------- ----------------------------------------------
  `prayer_times`        Calculated five prayer times
  `prayer_events`       Sunrise/Sunset/Sehri/Iftar calculated events
  `jamaat_settings`     Five user-defined Jamaat times
  `tahajjud_settings`   User-defined Tahajjud time
  `alarms`              Personal alarms
  `vibration_events`    Scheduled/triggered event ledger

### 11.3 Proposed SQL schema

নিচের schema একটি starting design। Production migration-এর আগে সব
nullable/unique/index rule test করতে হবে।

``` sql
PRAGMA foreign_keys = ON;

CREATE TABLE IF NOT EXISTS collections (
  id TEXT PRIMARY KEY,
  name_ar TEXT NOT NULL,
  name_en TEXT NOT NULL,
  compiler TEXT,
  source_reference TEXT,
  verification_status TEXT NOT NULL,
  content_revision INTEGER NOT NULL DEFAULT 1
);

CREATE TABLE IF NOT EXISTS books (
  id TEXT PRIMARY KEY,
  collection_id TEXT NOT NULL REFERENCES collections(id) ON DELETE CASCADE,
  book_number TEXT,
  name_ar TEXT,
  name_en TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_books_collection
ON books(collection_id);

CREATE TABLE IF NOT EXISTS chapters (
  id TEXT PRIMARY KEY,
  book_id TEXT NOT NULL REFERENCES books(id) ON DELETE CASCADE,
  chapter_number TEXT,
  name_ar TEXT,
  name_en TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_chapters_book
ON chapters(book_id);

CREATE TABLE IF NOT EXISTS hadiths (
  id TEXT PRIMARY KEY,
  collection_id TEXT NOT NULL REFERENCES collections(id),
  book_id TEXT REFERENCES books(id),
  chapter_id TEXT REFERENCES chapters(id),
  hadith_number TEXT NOT NULL,
  edition_name TEXT,
  publisher TEXT,
  edition_year INTEGER,
  text_ar TEXT NOT NULL,
  grade_label TEXT,
  grade_source TEXT,
  verification_status TEXT NOT NULL,
  verified_by TEXT,
  verified_at TEXT,
  content_revision INTEGER NOT NULL DEFAULT 1,
  UNIQUE(collection_id, edition_name, hadith_number)
);

CREATE INDEX IF NOT EXISTS idx_hadith_collection_number
ON hadiths(collection_id, hadith_number);

CREATE TABLE IF NOT EXISTS hadith_translations (
  id TEXT PRIMARY KEY,
  hadith_id TEXT NOT NULL REFERENCES hadiths(id) ON DELETE CASCADE,
  language_code TEXT NOT NULL,
  text TEXT NOT NULL,
  translator TEXT,
  license TEXT,
  UNIQUE(hadith_id, language_code, translator)
);

CREATE TABLE IF NOT EXISTS topics (
  id TEXT PRIMARY KEY,
  name_en TEXT NOT NULL,
  name_ar TEXT
);

CREATE TABLE IF NOT EXISTS hadith_topics (
  hadith_id TEXT NOT NULL REFERENCES hadiths(id) ON DELETE CASCADE,
  topic_id TEXT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,
  PRIMARY KEY(hadith_id, topic_id)
);

CREATE TABLE IF NOT EXISTS adhkar (
  id TEXT PRIMARY KEY,
  text_ar TEXT NOT NULL,
  source_reference TEXT,
  recommended_count INTEGER,
  verification_status TEXT NOT NULL,
  content_revision INTEGER NOT NULL DEFAULT 1
);

CREATE TABLE IF NOT EXISTS adhkar_translations (
  id TEXT PRIMARY KEY,
  dhikr_id TEXT NOT NULL REFERENCES adhkar(id) ON DELETE CASCADE,
  language_code TEXT NOT NULL,
  text TEXT NOT NULL,
  translator TEXT,
  license TEXT,
  UNIQUE(dhikr_id, language_code, translator)
);

CREATE TABLE IF NOT EXISTS adhkar_topics (
  dhikr_id TEXT NOT NULL REFERENCES adhkar(id) ON DELETE CASCADE,
  topic_id TEXT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,
  PRIMARY KEY(dhikr_id, topic_id)
);

CREATE TABLE IF NOT EXISTS sessions (
  id TEXT PRIMARY KEY,
  prayer_key TEXT NOT NULL,
  content_type TEXT NOT NULL CHECK(content_type IN ('hadith', 'dhikr')),
  start_at TEXT NOT NULL,
  end_at TEXT NOT NULL,
  status TEXT NOT NULL CHECK(status IN ('scheduled', 'active', 'completed', 'skipped', 'cancelled')),
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_sessions_start
ON sessions(start_at);

CREATE TABLE IF NOT EXISTS session_items (
  id TEXT PRIMARY KEY,
  session_id TEXT NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
  content_type TEXT NOT NULL CHECK(content_type IN ('hadith', 'dhikr')),
  content_id TEXT NOT NULL,
  position INTEGER NOT NULL DEFAULT 0
);

CREATE INDEX IF NOT EXISTS idx_session_items_session
ON session_items(session_id, position);

CREATE TABLE IF NOT EXISTS user_progress (
  id TEXT PRIMARY KEY,
  content_type TEXT NOT NULL CHECK(content_type IN ('hadith', 'dhikr')),
  content_id TEXT NOT NULL,
  status TEXT NOT NULL,
  completed_at TEXT,
  note TEXT,
  UNIQUE(content_type, content_id)
);

CREATE TABLE IF NOT EXISTS app_settings (
  key TEXT PRIMARY KEY,
  value_json TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS sync_meta (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS prayer_times (
  id TEXT PRIMARY KEY,
  local_date TEXT NOT NULL,
  timezone TEXT NOT NULL,
  prayer_key TEXT NOT NULL CHECK(prayer_key IN ('fajr','dhuhr','asr','maghrib','isha')),
  calculated_at TEXT NOT NULL,
  adhan_at TEXT NOT NULL,
  calculation_method TEXT NOT NULL,
  calculation_config_json TEXT NOT NULL,
  UNIQUE(local_date, timezone, prayer_key)
);

CREATE TABLE IF NOT EXISTS prayer_events (
  id TEXT PRIMARY KEY,
  local_date TEXT NOT NULL,
  timezone TEXT NOT NULL,
  event_key TEXT NOT NULL CHECK(event_key IN ('sunrise','sunset','sehri_end','iftar')),
  event_at TEXT NOT NULL,
  calculation_config_json TEXT NOT NULL,
  UNIQUE(local_date, timezone, event_key)
);

CREATE TABLE IF NOT EXISTS jamaat_settings (
  prayer_key TEXT PRIMARY KEY CHECK(prayer_key IN ('fajr','dhuhr','asr','maghrib','isha')),
  local_time TEXT NOT NULL,
  enabled INTEGER NOT NULL DEFAULT 1 CHECK(enabled IN (0,1)),
  updated_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS tahajjud_settings (
  id INTEGER PRIMARY KEY CHECK(id = 1),
  local_time TEXT NOT NULL,
  enabled INTEGER NOT NULL DEFAULT 1 CHECK(enabled IN (0,1)),
  updated_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS alarms (
  id TEXT PRIMARY KEY,
  hour INTEGER NOT NULL CHECK(hour BETWEEN 1 AND 12),
  minute INTEGER NOT NULL CHECK(minute BETWEEN 0 AND 59),
  period TEXT NOT NULL CHECK(period IN ('AM','PM')),
  repeat_type TEXT NOT NULL CHECK(repeat_type IN ('once','daily')),
  once_date TEXT,
  vibration_enabled INTEGER NOT NULL DEFAULT 1 CHECK(vibration_enabled IN (0,1)),
  enabled INTEGER NOT NULL DEFAULT 1 CHECK(enabled IN (0,1)),
  next_trigger_at TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS vibration_events (
  id TEXT PRIMARY KEY,
  source_system TEXT NOT NULL CHECK(source_system IN ('prayer_library','jamaat','tahajjud','personal_alarm')),
  event_key TEXT NOT NULL,
  trigger_at TEXT NOT NULL,
  duration_ms INTEGER NOT NULL,
  status TEXT NOT NULL CHECK(status IN ('scheduled','triggered','missed','cancelled')),
  platform_notification_id TEXT,
  created_at TEXT NOT NULL,
  UNIQUE(source_system, event_key, trigger_at)
);

CREATE INDEX IF NOT EXISTS idx_vibration_trigger
ON vibration_events(trigger_at, status);
```

### 11.4 Database relationship map

``` text
collections 1 ─── N books 1 ─── N chapters
     │
     └──────────── 1 ─── N hadiths
                          │
                          ├── 1 ─── N hadith_translations
                          └── N ─── N topics (via hadith_topics)

adhkar 1 ─── N adhkar_translations
   └── N ─── N topics (via adhkar_topics)

sessions 1 ─── N session_items
session_items ── content_type + content_id ──> hadiths OR adhkar
user_progress ── content_type + content_id ──> hadiths OR adhkar

prayer_times      [Prayer Time Library]
prayer_events     [Prayer Time Library]
jamaat_settings   [Manual Jamaat — independent]
tahajjud_settings[Manual Tahajjud — independent]
alarms            [Personal alarms]
vibration_events [Scheduled event ledger]
```

**Polymorphic relation note:** `session_items` এবং `user_progress`-এ
`content_type + content_id` রাখা হয়েছে। SQLite-এ এই pattern-এর জন্য
সরাসরি দুই টেবিলে conditional foreign key সম্ভব নয়। তাই repository/service
layer-এ content type অনুযায়ী target record আছে কি না validate করতে হবে।
বিকল্পভাবে `session_hadith_items` ও `session_dhikr_items` আলাদা join
table করা যায়।

### 11.5 Database migration

-   `schema_version` একটি migration table বা `PRAGMA user_version` দিয়ে
    track করুন।
-   প্রতিটি migration transaction-এর মধ্যে চালান।
-   পুরোনো user data destructiveভাবে drop করবেন না।
-   migration failure হলে rollback করুন।
-   App startup-এ migration complete না হওয়া পর্যন্ত repository access শুরু
    করবেন না।

------------------------------------------------------------------------

## 12. Application Architecture & Code Organization

``` text
deen/
├── app/
│   ├── (tabs)/
│   │   ├── index.tsx              # Home
│   │   ├── alarm.tsx              # Alarm list
│   │   └── settings.tsx           # Settings
│   ├── alarm/
│   │   └── edit/[id].tsx
│   ├── hadith/
│   │   ├── index.tsx
│   │   └── [id].tsx
│   ├── dhikr/
│   ├── sessions/
│   └── history/
├── src/
│   ├── database/
│   │   ├── db.ts
│   │   ├── schema.ts
│   │   ├── migrations/
│   │   └── repositories/
│   ├── services/
│   │   ├── prayerService.ts
│   │   ├── jamaatService.ts
│   │   ├── tahajjudService.ts
│   │   ├── alarmService.ts
│   │   ├── vibrationService.ts
│   │   ├── sessionService.ts
│   │   └── syncService.ts
│   ├── features/
│   │   ├── home/
│   │   ├── prayer/
│   │   ├── alarm/
│   │   ├── settings/
│   │   ├── hadith/
│   │   ├── dhikr/
│   │   ├── sessions/
│   │   └── history/
│   ├── stores/
│   ├── hooks/
│   ├── components/
│   ├── types/
│   └── utils/
├── assets/
├── app.json
├── package.json
└── README.md
```

### 12.1 Layer responsibilities

-   **UI/screens:** render state, capture user input; business rules
    এখানে ছড়িয়ে দেওয়া যাবে না।
-   **Feature hooks/stores:** screen state ও UI coordination।
-   **Services:** prayer calculation, event scheduling, alarm, sessions,
    sync-এর business logic।
-   **Repositories:** SQLite read/write।
-   **Database migrations:** schema evolution।
-   **Content validators:** remote JSON validation ও integrity।
-   **Native bridge (if needed):** Android exact alarm/vibration
    behavior।

### 12.2 Separation of services

`prayerService` কখনো `jamaat_settings` update করবে না।\
`jamaatService` কখনো `prayer_times` recalculate করবে না।\
`alarmService` personal alarm ছাড়া অন্য event-এর owner হবে না।\
`syncService` কেবল verified content tables ও `sync_meta` পরিচালনা করবে;
user settings/progress নয়।

------------------------------------------------------------------------

## 13. Scheduling & Reconciliation Logic

App-এর schedule শুধু একবার তৈরি করে রেখে দেওয়া যথেষ্ট নয়। Date, timezone,
location, settings বা permission পরিবর্তন হলে reconcile করতে হবে।

### 13.1 Prayer schedule refresh

Trigger conditions:

-   নতুন local date
-   location change
-   timezone change
-   calculation method change
-   app start / network-independent refresh

Process:

1.  Current local date ও timezone resolve।
2.  Prayer calculation config load।
3.  পাঁচ prayer time calculate।
4.  চার extra event calculate/derive।
5.  `prayer_times` ও `prayer_events` upsert।
6.  Prayer Library-এর event policy অনুযায়ী `vibration_events` reconcile।
7.  আগের তারিখের obsolete scheduled event cancel/mark করুন।

### 13.2 Manual Jamaat/Tahajjud schedule

-   Settings save হলে শুধু সংশ্লিষ্ট manual event regenerate হবে।
-   পাঁচ Jamaat event ও Tahajjud event-এর source আলাদা থাকবে।
-   Duration: 3000 ms (৩ seconds), confirmed policy অনুযায়ী।
-   Daily schedule-এর next local occurrence হিসাব করতে timezone/DST edge
    case সামলাতে হবে।

### 13.3 Personal alarm schedule

-   Alarm create/update/toggle হলে `alarmService` schedule reconcile
    করবে।
-   Once alarm-এর তারিখ ও time validate করতে হবে।
-   Daily alarm-এর next occurrence local timezone-এ হিসাব হবে।
-   Disable/delete হলে OS notification/scheduled trigger cancel করতে
    হবে।

### 13.4 Reconciliation principle

Database হলো intended schedule-এর source of truth; OS notification
registry হলো execution mechanism। App launch/permission return-এ
intended schedule ও OS scheduled notification মিলিয়ে reconcile করতে হবে।

------------------------------------------------------------------------

## 14. Privacy & Permissions

-   Location কেবল prayer time calculation-এর জন্য; user-এর অনুমতি ছাড়া
    location history তৈরি করা হবে না।
-   সম্ভব হলে coordinates device-এ local রাখুন।
-   GitHub content sync-এ personal data পাঠানো হবে না।
-   Notification permission স্পষ্টভাবে চাইতে হবে।
-   Exact alarm permission দরকার হলে কেন দরকার তা ব্যাখ্যা করতে হবে।
-   Vibration permission/device capability যাচাই করতে হবে।
-   Permission না থাকলে feature-এর সীমাবদ্ধতা UI-তে জানাতে হবে।
-   কোনো permission silently bypass করা যাবে না।

------------------------------------------------------------------------

## 15. Error Handling

  -----------------------------------------------------------------------
  Scenario                            Expected behavior
  ----------------------------------- -----------------------------------
  No internet                         Local SQLite content ও settings
                                      দিয়ে app চলবে

  GitHub unavailable                  Sync skip; existing content থাকবে

  Invalid JSON                        File reject; transaction commit হবে
                                      না

  Hash mismatch                       Download reject; local data অক্ষত

  Unsupported schema version          Sync stop; app পুরোনো valid content
                                      ব্যবহার করবে

  Location unavailable                Last valid coordinates বা manual
                                      location workflow; stale data label
                                      করা

  Notification permission denied      UI status দেখানো; permission চাইতে
                                      user action

  Exact alarm restricted              Exact timing guarantee না দেওয়া;
                                      system limitation দেখানো

  Timezone changed                    prayer/alarm schedule reconcile

  Database migration failed           rollback; error log; user data
                                      preserve
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 16. Testing Strategy

### 16.1 Unit tests

-   Prayer calculation adapter inputs/outputs
-   Jamaat time independence
-   Tahajjud time independence
-   Alarm repeat/once calculation
-   Session elapsed-time calculation
-   Manifest/schema validation
-   SHA-256 verification
-   Content upsert transaction
-   Event ID uniqueness/collision logic

### 16.2 Integration tests

-   SQLite migrations from each supported version
-   GitHub sync success/failure/rollback
-   Prayer schedule refresh after date/timezone change
-   Jamaat update without prayer library mutation
-   Alarm disable cancels scheduled trigger
-   App restart restores session remaining time

### 16.3 Device tests

বিভিন্ন Android version ও manufacturer device-এ পরীক্ষা:

-   App foreground
-   App background
-   App force-stop
-   Device reboot
-   Battery saver
-   Do Not Disturb
-   Notification permission off/on
-   Exact alarm permission off/on
-   Timezone/date manually changed
-   Device offline

------------------------------------------------------------------------

## 17. Development Roadmap

### Phase 1 --- Foundation

-   Expo project setup
-   Expo Router
-   System theme support
-   SQLite database + migrations
-   Three-tab navigation
-   English/Arabic typography and RTL support where needed

### Phase 2 --- Home & Prayer Library

-   Prayer calculation configuration
-   9 calculated library events
-   Event Box
-   10 fixed cards
-   selected card state and prayer detail table

### Phase 3 --- Manual Times & Alarm

-   Five Jamaat settings
-   Tahajjud setting
-   3-second vibration event scheduling
-   Personal alarm list and wheel picker
-   Android permission handling

### Phase 4 --- Content Library

-   Hadith schema
-   Dhikr schema
-   Verified seed content
-   Content browsing/detail screens

### Phase 5 --- GitHub Sync

-   Manifest
-   SHA-256
-   JSON validation
-   Transactional import
-   Versioned content updates

### Phase 6 --- Sessions & History

-   Session logic
-   Hadith/Dhikr assignment
-   Timestamp-based timer
-   Progress/history

### Phase 7 --- Reliability

-   Native Android scheduling/vibration as required
-   Device matrix tests
-   Battery/Doze/exact alarm behavior
-   Backup and migration tests

------------------------------------------------------------------------

## 18. Open Decisions --- Implementation-এর আগে নিশ্চিত করতে হবে

1.  **Prayer Time Library vibration:** ১ মিনিট vibration কি শুধু পাঁচ
    prayer-এ হবে, নাকি Tahajjud ও Sunrise/Sunset/Sehri/Iftar-সহ সব
    library event-এ হবে? পূর্বের নির্দেশে ভিন্নতা আছে।
2.  **Library-এর Tahajjud:** Library-এর ৯টি event-এর তালিকায় Tahajjud
    ছিল না, কিন্তু vibration আলোচনায় Tahajjud এসেছে। Tahajjud কি
    calculated library event, নাকি কেবল Settings-এর manual event?
3.  **Vibration collision:** একই সময়ে Adhan/library ও Jamaat event হলে
    vibration একত্র হবে, ধারাবাহিক হবে, নাকি priority থাকবে?
4.  **Countdown:** Home Event Box-এ Hijri date ও schedule নিশ্চিত। Next
    Jamaat/Adhan countdown-ও দেখাতে হবে কি না, তা পুনরায় নিশ্চিত করতে হবে।
5.  **Session content mapping:** Fajr/Dhuhr/Asr/Maghrib/Isha-তে কোনটি
    Hadith বা Dhikr হবে, এবং mapping user পরিবর্তন করতে পারবেন কি না।
6.  **Session UX:** Session start, pause, resume, skip, finish screen ও
    notification behavior।
7.  **History:** কোন activity, statistics ও retention period দেখানো হবে।
8.  **Location input:** GPS, manual city/coordinates, নাকি দুটোই।
9.  **Calculation method:** কোন prayer calculation method ও Asr madhab
    default হবে।
10. **Sehri/Iftar convention:** Fajr/Maghrib-এর সঙ্গে সরাসরি link থাকবে,
    নাকি configurable offset থাকবে।
11. **Once alarm:** trigger হওয়ার পর auto-disable হবে কি না।
12. **Hijri date:** device/local calculation, নাকি user-selectable Hijri
    adjustment থাকবে।

------------------------------------------------------------------------

## 19. Acceptance Criteria

প্রথম production-ready version গ্রহণযোগ্য হবে যখন:

-   Bottom navigation-এ ঠিক Home, Alarm, Settings থাকবে।
-   Home-এ ১টি Event Box + ঠিক ১০টি fixed card থাকবে।
-   Card tap করলে card size বদলাবে না; selected border green হবে।
-   Event Box selected card-এর তথ্য দেখাবে।
-   Prayer Library এবং manual Jamaat/Tahajjud database ও service layer-এ
    স্বাধীন থাকবে।
-   পাঁচ Jamaat + Tahajjud event-এ ৩ seconds vibration policy implement ও
    device-tested হবে।
-   কোনো event-এ music/ringtone থাকবে না।
-   Personal alarms আলাদা থাকবে।
-   App offline অবস্থায় local data দিয়ে চলবে।
-   GitHub sync শুধু validated content import করবে।
-   Hash mismatch/invalid JSON হলে local content অক্ষত থাকবে।
-   User settings/progress GitHub-এ upload হবে না।
-   UI-তে Bengali label থাকবে না; English ও Arabic ব্যবহৃত হবে।
-   Religious content source, translation license ও verification
    metadata ছাড়া verified হিসেবে প্রকাশ হবে না।

------------------------------------------------------------------------

## 20. চূড়ান্ত Architecture Summary

``` text
                    DEEN Android App
                           │
             ┌─────────────┴─────────────┐
             │                           │
          UI Layer                  Navigation
             │                   Home / Alarm / Settings
             │
        Feature Layer
             │
        Service Layer
   ┌─────────┼──────────────┬───────────────┐
   │         │              │               │
Prayer    Jamaat &        Alarm           Content
Service   Tahajjud       Service          Sync
   │      Service          │               │
   │         │              │          GitHub JSON
   │         │              │          + Manifest
   └─────────┴──────────────┴───────────────┘
                           │
                    Repository Layer
                           │
                       SQLite DB
                           │
                 Local source of truth
```

**সবচেয়ে গুরুত্বপূর্ণ নকশাগত সিদ্ধান্ত:** Prayer Time Library, Manual
Jamaat/Tahajjud এবং Personal Alarm---এই তিনটি scheduling domain আলাদা
থাকবে। তাদের event identity, settings, service, database ownership ও
schedule reconciliation আলাদা হবে। GitHub কেবল verified religious
content সরবরাহ করবে; user-specific data নয়।
