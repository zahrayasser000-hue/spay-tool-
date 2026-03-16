import streamlit as st
import google.generativeai as genai
import json
import re
import requests
import random
import urllib.parse

# --- إعدادات الصفحة ---
st.set_page_config(page_title="ALI Spy Pro - Product Radar", layout="wide", page_icon="🕵️‍♂️")

# --- التصميم العبقري (Premium UI/UX CSS) ---
st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;900&display=swap');
    
    html, body, [data-testid="stAppViewContainer"], [data-testid="stSidebar"] { 
        font-family: 'Cairo', sans-serif !important; direction: rtl; text-align: right; background-color: #f8fafc;
    }
    
    ::-webkit-scrollbar { width: 8px; height: 8px; }
    ::-webkit-scrollbar-track { background: #f1f5f9; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
    
    .main-header { 
        background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
        color: white; padding: 40px 20px; border-radius: 20px; text-align: center; margin-bottom: 35px; 
        box-shadow: 0 20px 40px -10px rgba(15, 23, 42, 0.3); border-bottom: 5px solid #ef4444; position: relative; overflow: hidden;
    }
    .main-header h1 {
        font-weight: 900; font-size: 3rem; margin-bottom: 5px;
        background: linear-gradient(to right, #fca5a5, #ffffff); -webkit-background-clip: text; -webkit-text-fill-color: transparent; position: relative; z-index: 1;
    }
    .main-header p { color: #94a3b8; font-size: 1.2rem; font-weight: 600; position: relative; z-index: 1; }

    .stButton > button {
        background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%) !important; color: white !important; font-weight: 800 !important;
        font-size: 1.1rem !important; border: none !important; border-radius: 12px !important; padding: 15px 30px !important;
        transition: all 0.3s ease !important; box-shadow: 0 4px 6px -1px rgba(239, 68, 68, 0.3) !important; width: 100%;
    }
    .stButton > button:hover { transform: translateY(-3px) scale(1.01) !important; box-shadow: 0 15px 25px -5px rgba(239, 68, 68, 0.4) !important; }

    /* بطاقات التجسس (Spy Cards) */
    .spy-card {
        background: white; border-radius: 16px; padding: 20px; border: 1px solid #e2e8f0;
        box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.05); transition: transform 0.3s ease; height: 100%; display: flex; flex-direction: column;
    }
    .spy-card:hover { transform: translateY(-5px); border-color: #ef4444; box-shadow: 0 20px 25px -5px rgba(239, 68, 68, 0.15); }
    .spy-badge { background: #fee2e2; color: #ef4444; padding: 5px 15px; border-radius: 20px; font-size: 0.85rem; font-weight: 900; display: inline-block; margin-bottom: 15px; }
    .spy-price { font-size: 1.8rem; font-weight: 900; color: #10b981; }
    .spy-cost { font-size: 1.1rem; font-weight: bold; color: #94a3b8; text-decoration: line-through; margin-right: 10px; }
    .spy-img-container { width: 100%; height: 220px; border-radius: 12px; overflow: hidden; margin-bottom: 15px; background: #f1f5f9;}
    .spy-img { width: 100%; height: 100%; object-fit: cover; }
    
    .stTextInput input, .stSelectbox > div > div {
        border-radius: 10px !important; border: 1px solid #cbd5e1 !important; transition: border-color 0.3s !important;
    }
    .stTextInput input:focus { border-color: #ef4444 !important; box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.2) !important; }
</style>
""", unsafe_allow_html=True)

st.markdown('<div class="main-header"><h1>🕵️‍♂️ ALI Spy Pro</h1><p>رادار المنتجات الرابحة | تجسس وتحليل للسوق العربي (COD)</p></div>', unsafe_allow_html=True)

# ==========================================================
# 🧠 دوال أساسية (توليد، تنظيف، جلب الصور)
# ==========================================================
def get_fast_working_model(api_key):
    if 'valid_model_name' in st.session_state:
        return st.session_state.valid_model_name
    genai.configure(api_key=api_key, transport="rest")
    try:
        for m in genai.list_models():
            if 'generateContent' in m.supported_generation_methods and 'flash' in m.name.lower():
                st.session_state.valid_model_name = m.name
                return m.name
    except: pass
    st.session_state.valid_model_name = "gemini-pro"
    return "gemini-pro"

def get_real_or_ai_image(keyword, pexels_key, width, height, orientation="landscape"):
    safe_keyword = str(keyword).strip()
    if not safe_keyword or safe_keyword.lower() == "none": safe_keyword = "ecommerce product"
    
    # 1. جلب صورة حقيقية من Pexels إذا توفر المفتاح
    if pexels_key:
        try:
            url = f"https://api.pexels.com/v1/search?query={urllib.parse.quote(safe_keyword)}&per_page=5&orientation={orientation}"
            headers = {"Authorization": pexels_key}
            response = requests.get(url, headers=headers, timeout=5)
            if response.status_code == 200:
                data = response.json()
                if data.get("photos") and len(data["photos"]) > 0:
                    return random.choice(data["photos"])["src"]["large"]
        except: pass
        
    # 2. الخيار البديل: صورة ذكاء اصطناعي بنمط واقعي
    prompt = f"professional ecommerce product photography of {safe_keyword}, white background, studio lighting, highly detailed 8k"
    seed = random.randint(1, 100000)
    return f"https://image.pollinations.ai/prompt/{urllib.parse.quote(prompt)}?width={width}&height={height}&nologo=true&seed={seed}"

def extract_json(text):
    tb = chr(96) * 3 
    clean_text = re.sub(f'{tb}(?:json|JSON)?', '', text, flags=re.IGNORECASE)
    clean_text = clean_text.replace(tb, '').strip()
    match = re.search(r'\[.*\]', clean_text, re.DOTALL) # نبحث عن مصفوفة Array
    if match: return match.group(0)
    match_obj = re.search(r'\{.*\}', clean_text, re.DOTALL)
    if match_obj: return f"[{match_obj.group(0)}]" # تغليف إذا أرجع عنصر واحد
    return clean_text

# ==========================================================
# 🕵️‍♂️ دالة رادار المنتجات (Spy Tool AI)
# ==========================================================
def generate_spy_products(api_key, market, niche):
    genai.configure(api_key=api_key, transport="rest")
    model_name = get_fast_working_model(api_key)
    model = genai.GenerativeModel(model_name)
    
    prompt = f"""
    أنت خوارزمية ذكية متخصصة في تحليل التجارة الإلكترونية (Drop-shipping & COD) تعمل كأداة تجسس قوية (مثل Minea أو SellTheTrend).
    قم بتحليل السوق الحالي في "{market}" وتحديداً في مجال "{niche}".
    
    استخرج 3 إلى 5 منتجات "فعلية ومحددة جداً" (وليس تصنيفات عامة مثل "ساعة ذكية" بل منتج مخصص مثل "ساعة ذكية لقياس ضغط الدم لكبار السن") تعتبر حالياً منتجات رابحة (Winning Products) وتحقق مبيعات ضخمة بنظام الدفع عند الاستلام (COD) في هذا البلد.

    رد بصيغة JSON Array بهذا الهيكل الدقيق حصراً:
    [
        {{
            "product_name": "اسم المنتج الدقيق بالعربية",
            "category": "Gadgets أو Cosmetics",
            "image_keyword": "1 to 3 english words describing the product strictly for image search",
            "why_winning": "سبب مقنع جداً لنجاحه في هذا البلد بالتحديد (سطر واحد)",
            "target_audience": "الجمهور المستهدف بدقة",
            "cost_price": 5, 
            "selling_price": 35
        }}
    ]
    لا تكتب أي نص أو شرح خارج مصفوفة الـ JSON.
    """
    response = model.generate_content(prompt, request_options={"timeout": 60.0})
    return extract_json(response.text)

# ==========================================================
# 🎛️ واجهة المستخدم
# ==========================================================

with st.sidebar:
    st.header("⚙️ إعدادات الرادار")
    api_key = st.text_input("🔑 Gemini API Key", type="password", help="مطلوب لتشغيل خوارزمية التجسس وتحليل السوق")
    pexels_key = st.text_input("📸 Pexels API Key (اختياري)", type="password", help="لجلب صور فوتوغرافية حقيقية للمنتجات بدلاً من الذكاء الاصطناعي")
    st.markdown("---")
    st.info("💡 **كيف تعمل الأداة؟**\nتقوم الأداة باستخدام الذكاء الاصطناعي للبحث في التريندات الحالية، الثقافة الشرائية للبلد المختار، واحتياجات السوق لاستخراج منتجات ذات احتمالية عالية جداً للنجاح في نظام الـ COD.")

st.markdown("### 📡 توجيه الرادار (تحديد الهدف)")
col1, col2 = st.columns(2)

with col1: 
    target_market = st.selectbox("🌍 السوق المستهدف (البلد):", [
        "المملكة العربية السعودية (KSA)", 
        "الإمارات العربية المتحدة (UAE)", 
        "المغرب (Morocco)", 
        "سلطنة عمان (Oman)",
        "الكويت (Kuwait)",
        "مصر (Egypt)",
        "الخليج العربي عموماً"
    ])
    
with col2: 
    target_niche = st.selectbox("🎯 النيتش (المجال):", [
        "منتجات حل المشاكل اليومية (Problem Solving Gadgets)", 
        "مستحضرات التجميل والعناية بالبشرة والشعر", 
        "أدوات المطبخ والمنزل الذكية", 
        "اكسسوارات وعناية السيارات", 
        "منتجات الصحة والراحة (آلام الظهر، المفاصل..)",
        "منتجات الأطفال والألعاب الذكية"
    ])

st.markdown("---")
spy_btn = st.button("📡 مسح السوق الآن (البحث عن منتجات رابحة)", use_container_width=True)

if spy_btn:
    if not api_key:
        st.error("⚠️ يرجى إدخال مفتاح Gemini API في القائمة الجانبية أولاً.")
    else:
        with st.spinner(f"🤖 جاري زحف وتحليل بيانات المستهلكين في {target_market}... (قد يستغرق 30 ثانية)"):
            try:
                raw_json = generate_spy_products(api_key, target_market, target_niche)
                products_list = json.loads(raw_json)
                st.session_state['spy_results'] = products_list
                st.success("✅ اكتمل المسح! إليك المنتجات التي تتصدر التريند حالياً.")
            except Exception as e:
                st.error(f"🛑 حدث خطأ أثناء تحليل السوق: تأكد من صحة الـ API Key أو حاول مرة أخرى.")
                st.write(f"تفاصيل الخطأ التقني: {str(e)}")

# ==========================================================
# 📊 عرض النتائج (Spy Cards)
# ==========================================================
if 'spy_results' in st.session_state and isinstance(st.session_state['spy_results'], list):
    st.markdown("<br>", unsafe_allow_html=True)
    st.markdown("### 🔥 المنتجات الرابحة (Winning Products)")
    
    # تنسيق العرض في أعمدة بناءً على عدد المنتجات المستخرجة
    num_products = len(st.session_state['spy_results'])
    cols = st.columns(min(num_products, 3)) # عرض 3 منتجات في الصف كحد أقصى
    
    for i, prod in enumerate(st.session_state['spy_results']):
        col_index = i % 3
        with cols[col_index]:
            st.markdown('<div class="spy-card">', unsafe_allow_html=True)
            
            # جلب الصورة
            keyword = prod.get('image_keyword', prod.get('product_name', 'product'))
            img_url = get_real_or_ai_image(keyword, pexels_key, 400, 300)
            
            st.markdown(f'''
            <div class="spy-img-container">
                <img src="{img_url}" class="spy-img" alt="{prod.get('product_name', 'Product')}" loading="lazy">
            </div>
            ''', unsafe_allow_html=True)
            
            st.markdown(f"<span class='spy-badge'>{prod.get('category', 'Gadgets')}</span>", unsafe_allow_html=True)
            st.markdown(f"<h3 style='font-weight:900; color:#0f172a; margin-top:0;'>{prod.get('product_name', 'بدون اسم')}</h3>", unsafe_allow_html=True)
            
            st.markdown("---")
            st.markdown(f"**🎯 الجمهور المستهدف:**<br><span style='color:#475569;'>{prod.get('target_audience', '')}</span>", unsafe_allow_html=True)
            st.markdown(f"**💡 سر النجاح في السوق:**<br><span style='color:#475569;'>{prod.get('why_winning', '')}</span>", unsafe_allow_html=True)
            st.markdown("---")
            
            st.markdown(f'''
            <div style="display:flex; justify-content:space-between; align-items:center; margin-top:auto; padding-top:15px;">
                <div>
                    <div style="font-size:0.9rem; color:#64748b; font-weight:bold;">تكلفة المورد</div>
                    <div class="spy-cost">${prod.get('cost_price', '0')}</div>
                </div>
                <div style="text-align:left;">
                    <div style="font-size:0.9rem; color:#64748b; font-weight:bold;">سعر البيع (COD)</div>
                    <div class="spy-price">${prod.get('selling_price', '0')}</div>
                </div>
            </div>
            ''', unsafe_allow_html=True)
            
            st.markdown('</div>', unsafe_allow_html=True)
            st.write("<br>", unsafe_allow_html=True)
