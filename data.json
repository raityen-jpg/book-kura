import os
import json
import re
from datetime import datetime
from google import genai

# 環境変数からAPIキーとアソシエイトIDを取得
API_KEY = os.environ.get("GEMINI_API_KEY")
TRACKING_ID = os.environ.get("AMAZON_TRACKING_ID", "manga-kura-22")

client = genai.Client(api_key=API_KEY)

prompt = """
あなたは書店の名物バイヤー・書評家です。
現在Kindleでセール・ポイント還元中、または話題となっている漫画・書籍の中から、
読者に自信を持っておすすめできる作品を厳選し、以下のJSON形式で出力してください。

【厳選条件】
- カテゴリは以下の3種類からバランスよく計6〜9作品選定してください:
  1. "50pct"（50%以上ポイント還元、実質半額など）
  2. "free"（1〜3巻無料、または大幅割引キャンペーン中）
  3. "completed"（全巻完結名作の一気読み、または珠玉の名短編集）
- 見どころ（highlight）は、あらすじだけでなく「なぜ大人が今読むべきか」「何が心を揺さぶるのか」を熱量のある書店POP風に50〜80文字程度で簡潔に記述してください。

【出力JSONフォーマット】
{
  "items": [
    {
      "title": "作品名（巻数指定がある場合はそれも）",
      "author": "著者名",
      "category": "50pct" または "free" または "completed",
      "badge": "50%還元" や "1-3巻無料" などの短いバッジ表記,
      "badge_color": "bg-amber-600"（還元）または "bg-emerald-600"（無料）または "bg-indigo-600"（完結）,
      "tags": "#少年コミック #完結 など",
      "sale_price": "実質 ○○円 または 無料 または ○○円",
      "original_price": "定価表記（あれば）",
      "search_query": "Amazon検索用キーワード（例: ダンジョン飯 1巻 Kindle）",
      "image_url": "書影画像のURL（不明な場合は空文字）",
      "highlight": "書店POP風の見どころ解説文"
    }
  ]
}
※マークダウン記法（```json など）は含めず、純粋なJSON文字列のみを出力してください。
"""

try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt,
    )
    
    text = response.text.strip()
    # コードブロックが含まれていた場合のクリーンアップ
    text = re.sub(r"^```json\s*", "", text)
    text = re.sub(r"^```\s*", "", text)
    text = re.sub(r"\s*```$", "", text)

    data = json.loads(text)

    # AmazonアソシエイトURLの生成付与
    for item in data.get("items", []):
        query = item.get("search_query", item["title"])
        encoded_query = requests_quote = __import__('urllib.parse').parse.quote(query)
        item["url"] = f"https://www.amazon.co.jp/s?k={encoded_query}&i=digital-text&tag={TRACKING_ID}"
        if not item.get("image_url"):
            # デフォルトプレースホルダー書影
            item["image_url"] = "https://images.unsplash.com/photo-1544716278-ca5e3f4abd8c?w=300&q=80"

    # 更新日時をJSTで記録
    now = datetime.now()
    data["updated_at"] = now.strftime("%Y/%m/%d %H:%M")

    with open("data.json", "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

    print("data.json successfully generated!")

except Exception as e:
    print(f"Error occurred: {e}")
    raise e
