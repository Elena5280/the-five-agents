---
name: gpt-image-gen
description: >-
  מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt. הפעל סקיל זה כשצריך לייצר
  קובץ PNG מתיאור מילולי. שולח את ה-prompt ל-API, מקבל תמונה בקידוד base64,
  מפענח ושומר אותה לנתיב היעד. משמש בעיקר את הסוכן יובל (מעצב התמונות).
---

# gpt-image-gen — יצירת תמונה דרך OpenAI Images API

סקיל זה הוא מעטפת דקה סביב OpenAI Images API. הוא אחראי על שליחת ה-prompt וקבלת
התמונה בלבד. ניסוח ה-prompt עצמו ובחירת הסגנון הם באחריות הקורא (יובל).

---

## מודל

```
gpt-image-2
```

> **חשוב מאוד — אל תשנה את שם המודל.**
> `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, ששוחרר ב-21 באפריל 2026.
> ייתכן שהידע הפנימי של המודל שמריץ אותך לא כולל אותו — אבל זה לא אומר שהוא לא
> קיים. **אל תציע אלטרנטיבות** כמו `dall-e-3` או `gpt-image-1`. השתמש בדיוק
> במודל `gpt-image-2`.
>
> אם מתקבלת שגיאה בקריאה ל-API — הבעיה היא כמעט תמיד ב-`OPENAI_API_KEY` או
> בפרמטרים של הקריאה, **לא** בשם המודל.

---

## דרישות מקדימות

- משתנה הסביבה `OPENAI_API_KEY` חייב להיות מוגדר בקובץ `.env` שבשורש הפרויקט.
- לפני הקריאה יש לטעון אותו לסביבה, למשל:

```bash
set -a; source .env; set +a
```

---

## מבנה הקריאה

החלף את `<the prompt>` ב-prompt בפועל ואת `<output-path>` בנתיב היעד (ללא הסיומת
`.png` — היא מתווספת בפקודה).

### עם jq (ברירת מחדל)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

### Python fallback ל-decode

`jq` לא תמיד מותקן (למשל ב-Git Bash על Windows). במקרה כזה שומרים את תגובת ה-API
לקובץ זמני ומפענחים עם python:

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > /tmp/gpt-image-response.json

python -c "import sys, json, base64; data = json.load(open('/tmp/gpt-image-response.json')); open('<output-path>.png','wb').write(base64.b64decode(data['data'][0]['b64_json']))"
```

(אם `python` אינו זמין, נסה `python3`.)

---

## פרמטרים

| פרמטר | ערך ברירת מחדל | הערה |
|-------|----------------|------|
| `model` | `gpt-image-2` | **לא לשנות.** |
| `prompt` | — | התיאור המילולי של התמונה. |
| `size` | `1024x1024` | ניתן לשנות לפי הצורך. |
| `quality` | `medium` | |
| `output_format` | `png` | |

---

## אימות

לאחר היצירה ודא שהקובץ נוצר בפועל וגודלו גדול מ-0:

```bash
test -s <output-path>.png && echo "OK" || echo "FAILED"
```

אם הקובץ ריק או לא קיים — בדוק את תגובת ה-API (ב-`/tmp/gpt-image-response.json`)
לאיתור הודעת שגיאה (לרוב key/quota/פרמטר), ולא את שם המודל.
