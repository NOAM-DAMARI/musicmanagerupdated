# מה האפליקציה שולחת ל-GitHub ומה היא קוראת משם

## 1. הגדרות החיבור (נשמרות במכשיר בלבד – לא עולות ל-GitHub)
| הגדרה | ברירת מחדל | איפה בקוד |
|---|---|---|
| בעלים (owner) | `NOAM-DAMARI` | `lib/services/app_state.dart` – `githubOwner` |
| מאגר (repo) | `musicmanagerupdated` | `githubRepo` |
| ענף (branch) | `main` | `githubBranch` |
| Token | – (חובה) | `githubToken`, נשמר ב-SharedPreferences של המכשיר |

- ה-Token צריך הרשאת כתיבה לתוכן המאגר (Classic PAT עם `repo`, או Fine-grained עם **Contents: Read and write**).
- כל בקשה נשלחת עם: `Authorization: Bearer <token>`, `Accept: application/vnd.github+json`, `X-GitHub-Api-Version: 2022-11-28`.
- **לא עולים ל-GitHub:** ה-Token, מפתחות מנועי התמלול (`tr_key_*`), מצב כהה, נתיב התיקייה המקומית, מטמון קבצים לא-מקוון, מיקום האזנה (`pos_*`).

## 2. המבנה שחייב להיות במאגר
```
<repo>/
├─ data/
│   └─ music_manager.json          ← מסד הנתונים היחיד (נקרא וגם נכתב)
├─ media/
│   ├─ lessons_<זמן>_<שם>.mp3      ← קבצי שיעורים קטנים (Contents API)
│   ├─ songs_<זמן>_<שם>.mp3        ← קבצי שירים קטנים
│   ├─ inbox_<זמן>_<שם>.mp3        ← קבצים "לא משויכים" קטנים
│   └─ singers/<זמן>_<שם>.jpg      ← תמונות זמרים
└─ Release בשם התגית: music-manager-media
    └─ assets: lessons_/songs_/inbox_<זמן>_<שם>   ← קבצים גדולים (עד 2GB)
```
- **אין צורך ליצור כלום ידנית.** התיקיות נוצרות בהעלאה הראשונה (GitHub יוצר תיקייה כשכותבים קובץ בנתיב שלה), וה-Release נוצר אוטומטית בהעלאת הקובץ הגדול הראשון (`GitHubMedia._ensureRelease`).
- שם הקובץ במאגר הוא שם טכני (אותיות לועזיות בלבד – תווים אחרים הופכים ל-`_`). השם המקורי (גם בעברית) נשמר בשדה `fileName` ב-JSON, ובהורדה הקובץ יורד בשם המקורי.
- המאגר יכול להיות פרטי. הורדה/השמעה ממאגר פרטי נעשית דרך ה-API עם ה-Token.

## 3. קובץ ה-JSON: `data/music_manager.json`
מפתחות ברמה העליונה (נבנים ב-`AppState.export()`, נטענים ב-`AppState.restore()`):

| מפתח | תוכן |
|---|---|
| `singers` | `id, name, notes, image, color` |
| `songs` | `id, title, singerId, lyrics, source, notes, year, duration, fav, stamps, added, eventDate, location` + שדות מדיה |
| `lessons` | `id, title, parasha, parashaKind, number, dateKind, date, notes, summary, fav, added, points, pointNotes, pointTimes, pointEnds, pointCats, pointSubs, marks, transcript, transcripts` + שדות מדיה |
| `playlists` | `id, name, songIds` |
| `customParashot` | פרשות/אירועים שהוספת ידנית |
| `unassignedMedia` | קבצים שהועלו ועדיין לא שויכו (`id, added, lessonInfo` + שדות מדיה) |
| `importedLessonInfo` | פרטי שיעור מוצעים מייבוא ZIP |
| `_updatedAt` | חותמת זמן (UTC) של השינוי האחרון – קובעת איזה עותק חדש יותר |

**שדות מדיה** (בכל שיר / שיעור / לא משויך): `filePath` (נתיב מקומי במכשיר – לא תקף במכשירים אחרים), `fileName` (שם מקורי), `remoteUrl` (כתובת הורדה), `assetApi` (כתובת ה-API של הקובץ – משמשת להורדה ממאגר פרטי ולמחיקה), `fileSize`.

> שים לב: `filePath` נשמר גם הוא בקובץ ה-JSON שעולה למאגר, כלומר נתיב מקומי של המחשב/טלפון שלך נחשף בתוכו. אין בו סיסמאות.

## 4. מה נשלח ומתי

### א. שמירת הנתונים (JSON)
1. כל שינוי באפליקציה → `app.save()` מעדכן `_updatedAt`.
2. כתיבה לדיסק המקומי אחרי **0.4 שניות** של שקט.
3. העלאה ל-GitHub אחרי **2 שניות** של שקט (`_queuePush` → `syncNow`).
4. הבקשות (`lib/services/storage_provider.dart`):
   - **קריאה:** `GET /repos/{owner}/{repo}/contents/data/music_manager.json?ref={branch}` → התשובה כוללת `content` (base64) ו-`sha`.
   - **כתיבה:** `PUT` לאותו נתיב עם `message`, `content` (base64 של כל ה-JSON), `branch`, ו-`sha` של הגרסה הקיימת (אם הקובץ כבר קיים).
5. בכל כתיבה נשלח **הקובץ כולו**, לא שינויים בלבד.

### ב. סנכרון (`AppState.syncNow`)
- נקרא בעת הפעלה, בלחיצה על "סנכרון עכשיו" בהגדרות ואחרי שמירה.
- קורא את הנתונים מכל המקורות הפעילים (GitHub ו/או תיקייה מקומית `<תיקייה>/data/music_manager.json`) ומשווה לנתונים שבמכשיר.
- **העותק עם `_updatedAt` החדש ביותר מנצח** והוא נטען למכשיר, ואז נכתב לכל המקורות הפעילים.
- **אין מיזוג ברמת שדה או שיעור.** אם ערכת במכשיר א׳ ובמכשיר ב׳ בלי סנכרון ביניהם – השמירה האחרונה דורסת את השנייה. מומלץ לסנכרן לפני תחילת עבודה במכשיר אחר.
- מחיקות: נתונים שנמחקו באפליקציה נמחקים מהקובץ בסנכרון הבא (כי הקובץ נכתב מחדש כולו).

### ג. העלאת קובץ שמע/וידאו (`lib/services/github_media.dart`)
שם הנכס: `<תיקייה>_<זמן-במילישניות>_<שם-ללא-תווים-מיוחדים>`, כאשר התיקייה היא `lessons` / `songs` / `inbox`.

| גודל הקובץ | שיטה | יעד |
|---|---|---|
| עד 8MB באפליקציה / עד 25MB בדפדפן | Contents API: `PUT /contents/media/<שם הנכס>` (base64) | `media/` בתוך הענף |
| גדול מזה (או ש-GitHub דחה כ"גדול מדי") | Release asset: `POST uploads.github.com/repos/{repo}/releases/{id}/assets?name=<שם הנכס>` | Release `music-manager-media` |

- הגבלה: 2GB לקובץ.
- בסיום ההעלאה נשמרים ב-JSON: `remoteUrl`, `assetApi`, `fileName`, `fileSize`.
- החלפת קובץ מוחקת את הנכס הישן.
- תמונות זמרים: `PUT /contents/media/singers/<זמן>_<שם>` (`AppState.uploadMediaBytes`).

### ד. מחיקת קובץ (`GitHubMedia.deleteAsset`)
- קובץ מ-Contents API (הכתובת כוללת `/contents/`): קודם `GET` לקבלת `sha`, ואז `DELETE` עם `message`, `sha`, `branch`.
- Release asset: `DELETE` פשוט (תשובה 204).
- קובץ שכבר לא קיים (404) נחשב כנמחק.

### ה. קריאת (הורדת/השמעת) קבצים (`media_resolver.dart`, `media_download.dart`)
סדר חיפוש הקובץ:
1. קובץ מקומי (`filePath`) אם קיים במכשיר.
2. עותק במטמון (נשמר לפי גיבוב של הכתובת + שם הקובץ).
3. `assetApi` עם ה-Token (`Accept: application/vnd.github.raw+json` לקובץ מ-Contents, `application/octet-stream` ל-Release asset).
4. `remoteUrl` (עם ה-Token אם זה `raw.githubusercontent.com`, ואם נכשל – בלי).

## 5. מה האפליקציה לא עושה
- לא יוצרת מאגר, ענף או Token – את אלה אתה מכין ב-GitHub.
- לא משתמשת ב-Git (clone/commit) – רק ב-REST API.
- לא שומרת היסטוריית גרסאות משלה: ההיסטוריה היא ה-commits שנוצרים בכל כתיבה לקובץ ה-JSON (אפשר לשחזר גרסה ישנה מ-GitHub, בלשונית ה-History של הקובץ).

## 6. סנכרון מול תיקייה מקומית (במקום או לצד GitHub)
מבנה התיקייה המקומית:
```
<התיקייה שבחרת>/
├─ data/music_manager.json
└─ media/<תת-תיקיות>/<זמן>_<שם>
```
- המעבר בין GitHub / תיקייה / שניהם (בהגדרות) אינו מוחק Token, שם מאגר או נתיב תיקייה.
- אם התיקייה המקומית היא תיקייה שמסונכרנת עם Google Drive – הנתונים יסונכרנו גם לשם.
