# מפת הפרויקט – איפה כל דבר נמצא

כל הקוד ב-`lib/`. נקודת הכניסה: `lib/main.dart`.

## מסמכים בשורש
| קובץ | תוכן |
|---|---|
| `GITHUB_STRUCTURE.md` | מה נשלח ל-GitHub, מה נקרא משם, ומבנה המאגר |
| `PLATFORM_SETUP.md` | הקמת הפרויקט והרשאות לפי פלטפורמה |
| `CHANGELOG_V2.*.md`, `CHANGES_V1.md` | היסטוריית שינויים |
| `setup.bat` / `setup.sh` | יצירת תיקיות הפלטפורמה (`flutter create`) |
| `pubspec.yaml` | חבילות וגרסה |
| `web/index.html` | דף הפתיחה באתר + מסך הטעינה (התמונה מוטמעת בו) |
| `assets/fonts/` | גופנים ל-PDF (Noto Sans Hebrew + DejaVu למספרים) |
| `test/` | בדיקות |

## מסכים – `lib/screens/`
| קובץ | מה יש בו |
|---|---|
| `shell.dart` | סרגל הניווט הראשי (שירים / שיעורים / קבצים / הגדרות) |
| `dashboard_screen.dart` | ראשי – שירים |
| `singers_screen.dart`, `singer_page.dart`, `singer_form.dart` | זמרים: רשימה, עמוד זמר, הוספה/עריכה |
| `songs_screen.dart`, `song_form.dart`, `song_player_page.dart` | שירים: רשימה, עריכה, נגן מילים |
| `playlists_screen.dart` | רשימות השמעה |
| `lessons_dashboard_screen.dart` | ראשי – שיעורים |
| `lessons_screen.dart` | חיפוש ורשימת שיעורים (מיון + PDF) |
| `lesson_form.dart` | הוספה/עריכת שיעור (שם, מספר, פרשה, תאריך, קובץ) |
| `lesson_page.dart` | **עמוד שיעור**: נגן, כתוביות, סימניות, סיכום, נקודות |
| `subtitle_editor_screen.dart` | מצב עריכת כתוביות במקלדת |
| `transcripts_screen.dart` | תמלולים מרובים, גרסאות והשוואה |
| `points_screen.dart` | כל הנקודות מכל השיעורים (סינון, מיון, הדפסה) |
| `files_screen.dart` | ניהול קבצים: העלאה, "לא משויך", הורדה, מחיקה, החלפה |
| `zip_import_dialog.dart` | **טבלת ייבוא ZIP** (העמודות משתנות לפי סוג: שיעור / שיר / לא משויך) |
| `pdf_preview_screen.dart` | תצוגה מקדימה של PDF: הדפסה, הורדה, שיתוף |
| `settings_screen.dart` | הגדרות: מראה, GitHub, תיקייה מקומית, מקור אחסון, תמלול, גיבוי |
| `offline_models_screen.dart` | מודלי תמלול מקומיים (אנדרואיד) |

## שירותים – `lib/services/`
| קובץ | תפקיד |
|---|---|
| `app_state.dart` | **הלב**: כל הנתונים, שמירה, סנכרון, הגדרות GitHub, מקורות אחסון |
| `storage_provider.dart` | קריאה/כתיבה של ה-JSON: GitHub ותיקייה מקומית |
| `storage.dart` + `storage_io/stub.dart` | אחסון והמטמון במכשיר (ובדפדפן) |
| `github_media.dart` | העלאה ומחיקה של קבצי מדיה ב-GitHub |
| `media_upload.dart` | בחירת קובץ, צירוף לשיעור/שיר, בחירה מ"לא משויך" |
| `media_resolver.dart`, `media_download.dart` | איתור קובץ (מקומי/מטמון/ענן) והורדה בשם המקורי |
| `file_service.dart` | בחירת קבצים מהמכשיר |
| `folder_tools*.dart`, `local_sync*.dart` | עבודה עם תיקייה מקומית וקבצי ZIP בה |
| `zip_bytes.dart`, `zip_import_flow.dart` | קריאת ZIP/TAR ותהליך הייבוא המלא |
| `player_service.dart` | הנגן: מהירות, עוצמה, דילוגים, מיקום אחרון |
| `transcriber.dart` | מנועי תמלול (OpenAI, Groq, Soniox, מותאם) וטיפול בשגיאות (429 ועוד) |
| `text_aligner.dart` | חיבור טקסט מועלה לזמני התמלול (בלי לשנות אותיות) |
| `transcript_compare.dart` | השוואת תמלולים |
| `subtitle_formats.dart` | ייצוא/קריאת SRT, VTT, LRC, TXT |
| `export_service.dart` | PDF ויצוא: שיעור, נקודות (**רוחב עמודת הדקות** כאן), רשימות, גיבוי |
| `pdf_fonts.dart` | טעינת גופני ה-PDF |
| `points.dart` | נקודות, נושאים ותתי-נושאים |
| `platform_audio*.dart` | מנוע שמע ל-Windows/Linux |

## רכיבים, מודלים ועזרים
| תיקייה / קובץ | תוכן |
|---|---|
| `lib/widgets/player_controls.dart` | פקדי הנגן (מהירות, עוצמה, דילוג) |
| `lib/widgets/live_subtitles.dart` | כתוביות רצות |
| `lib/widgets/mini_player.dart` | נגן קטן קבוע |
| `lib/widgets/point_dialog.dart` | חלון הוספה/עריכת נקודה |
| `lib/widgets/parasha_picker.dart`, `hebrew_date_picker.dart` | בחירת פרשה ותאריך עברי/לועזי |
| `lib/widgets/transcription_settings.dart` | הגדרות מנועי תמלול + "בדיקת חיבור" |
| `lib/widgets/ui.dart`, `lesson_tile.dart`, `song_tile.dart`, `seg_list.dart` | רכיבי תצוגה |
| `lib/models/` | `singer`, `song`, `lesson` (+כתוביות), `playlist`, `media` (שדות מדיה), `unassigned_media`, `lesson_import_info` |
| `lib/utils/` | `helpers`, `parashot`, `hebrew_calendar`, `mp3` (זיהוי משך) |
| `lib/theme/app_theme.dart` | עיצוב בהיר/כהה |

## "איפה משנים…?"
| רוצה לשנות | קובץ |
|---|---|
| עמודות ה-PDF של הנקודות | `services/export_service.dart` – `buildPointsPdf` |
| העמודות בטבלת ייבוא ZIP | `screens/zip_import_dialog.dart` – `_visibleCols` |
| תמונת הפתיחה באתר | `web/index.html` (מוטמעת) ו-`web/splash.jpg` |
| ברירת המחדל של המאגר ב-GitHub | `services/app_state.dart` – `githubOwner/githubRepo/githubBranch` |
| מתי נשמר ומתי עולה ל-GitHub | `services/app_state.dart` – `save()` |
| מגבלת גודל להעלאה | `services/github_media.dart` – `contentsLimit`, `maxBytes` |
| רשימת הנושאים של הנקודות | `services/points.dart` – `pointCategories` |
| רשימת הפרשות | `utils/parashot.dart` |
