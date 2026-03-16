import React, { useState, useEffect } from 'react';
import { Search, Globe, Target, Image as ImageIcon, TrendingUp, DollarSign, Loader2, AlertTriangle, Heart, MessageCircle, Activity, ExternalLink, ShoppingBag, ArrowRight, SlidersHorizontal } from 'lucide-react';

// يتم توفير مفتاح Gemini تلقائياً من بيئة التشغيل
const apiKey = ""; 

// دالة جلب الصور (Pexels أو الذكاء الاصطناعي)
const fetchImage = async (keyword, pexelsKey) => {
  if (pexelsKey && pexelsKey.trim() !== "") {
    try {
      const res = await fetch(`https://api.pexels.com/v1/search?query=${encodeURIComponent(keyword)}&per_page=1&orientation=landscape`, {
        headers: { Authorization: pexelsKey }
      });
      const data = await res.json();
      if (data.photos && data.photos.length > 0) {
        return data.photos[0].src.large;
      }
    } catch (e) {
      console.error("فشل جلب الصورة من Pexels، سيتم استخدام البديل", e);
    }
  }
  // البديل: صورة ذكاء اصطناعي واقعية
  const prompt = `professional ecommerce product photography of ${keyword}, white background, studio lighting, high quality`;
  return `https://image.pollinations.ai/prompt/${encodeURIComponent(prompt)}?width=600&height=400&nologo=true&seed=${Math.floor(Math.random() * 100000)}`;
};

// مكون بطاقة المنتج
const ProductCard = ({ product, pexelsKey }) => {
  const [imgUrl, setImgUrl] = useState("");

  useEffect(() => {
    const getImg = async () => {
      const url = await fetchImage(product.image_keyword || product.product_name, pexelsKey);
      setImgUrl(url);
    };
    getImg();
  }, [product, pexelsKey]);

  const getSaturationColor = (level) => {
    if (level.includes("منخفض") || level.includes("Low")) return "bg-emerald-100 text-emerald-700 border-emerald-200";
    if (level.includes("متوسط") || level.includes("Medium")) return "bg-amber-100 text-amber-700 border-amber-200";
    return "bg-red-100 text-red-700 border-red-200";
  };

  return (
    <div className="bg-white rounded-2xl border border-slate-200 shadow-lg hover:-translate-y-2 hover:shadow-2xl transition-all duration-300 flex flex-col h-full overflow-hidden group">
      
      <div className="w-full h-56 bg-slate-100 overflow-hidden relative">
        {imgUrl ? (
          <img src={imgUrl} alt={product.product_name} className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500" loading="lazy" />
        ) : (
          <div className="w-full h-full flex items-center justify-center text-slate-400">
            <Loader2 className="w-8 h-8 animate-spin" />
          </div>
        )}
        <div className="absolute top-3 right-3 bg-slate-900/80 backdrop-blur-sm text-white px-3 py-1 rounded-full text-xs font-black shadow-sm">
          {product.category}
        </div>
        <div className={`absolute top-3 left-3 px-3 py-1 rounded-full text-xs font-black shadow-sm border ${getSaturationColor(product.saturation)}`}>
          التشبع: {product.saturation}
        </div>
      </div>
      
      <div className="p-5 flex flex-col flex-grow">
        <h3 className="font-black text-xl text-slate-800 mb-4 leading-tight">{product.product_name}</h3>
        
        <div className="flex items-center justify-center gap-4 mb-4 bg-slate-50 p-3 rounded-xl border border-slate-100">
          <div className="flex items-center gap-1.5 text-slate-700 font-bold text-sm">
            <Heart className="w-4 h-4 text-red-500 fill-red-500" />
            {product.engagement_likes}
          </div>
          <div className="w-px h-4 bg-slate-300"></div>
          <div className="flex items-center gap-1.5 text-slate-700 font-bold text-sm">
            <MessageCircle className="w-4 h-4 text-blue-500 fill-blue-500" />
            {product.engagement_comments}
          </div>
          <div className="w-px h-4 bg-slate-300"></div>
          <div className="flex items-center gap-1.5 text-emerald-600 font-black text-sm">
            <Activity className="w-4 h-4" />
            Trend
          </div>
        </div>

        <div className="grid grid-cols-3 gap-2 mb-5">
          <div className="bg-slate-50 p-2 rounded-lg text-center border border-slate-100">
            <div className="text-[10px] text-slate-500 font-bold mb-1">سعر المورد</div>
            <div className="text-slate-800 font-black text-sm">${product.cost_price}</div>
          </div>
          <div className="bg-emerald-50 p-2 rounded-lg text-center border border-emerald-100">
            <div className="text-[10px] text-emerald-600 font-bold mb-1">سعر البيع</div>
            <div className="text-emerald-700 font-black text-sm">${product.selling_price}</div>
          </div>
          <div className="bg-blue-50 p-2 rounded-lg text-center border border-blue-100">
            <div className="text-[10px] text-blue-600 font-bold mb-1">هامش الربح</div>
            <div className="text-blue-700 font-black text-sm">${product.profit_margin}</div>
          </div>
        </div>
        
        <div className="text-sm text-slate-600 mb-4">
          <strong className="text-slate-800">💡 لماذا ينجح؟</strong> {product.why_winning}
        </div>
        
        <div className="mt-auto grid grid-cols-2 gap-2 pt-4 border-t border-slate-100">
          <a 
            href={`https://www.facebook.com/ads/library/?active_status=all&ad_type=all&country=ALL&q=${encodeURIComponent(product.fb_search_query)}`}
            target="_blank" 
            rel="noopener noreferrer"
            className="flex items-center justify-center gap-2 bg-blue-600 hover:bg-blue-700 text-white py-2.5 rounded-xl text-xs font-bold transition-colors"
          >
            <Search className="w-3.5 h-3.5" />
            إعلانات فيسبوك
          </a>
          <a 
            href={`https://www.aliexpress.com/wholesale?SearchText=${encodeURIComponent(product.aliexpress_query)}`}
            target="_blank" 
            rel="noopener noreferrer"
            className="flex items-center justify-center gap-2 bg-orange-500 hover:bg-orange-600 text-white py-2.5 rounded-xl text-xs font-bold transition-colors"
          >
            <ShoppingBag className="w-3.5 h-3.5" />
            علي إكسبريس
          </a>
        </div>
      </div>
    </div>
  );
};

export default function App() {
  const [market, setMarket] = useState("المملكة العربية السعودية (KSA)");
  const [niche, setNiche] = useState("منتجات حل المشاكل اليومية");
  const [productCount, setProductCount] = useState(4); // المتغير الجديد لعدد المنتجات
  const [pexelsKey, setPexelsKey] = useState("");
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState([]);
  const [error, setError] = useState("");

  const markets = ["المملكة العربية السعودية (KSA)", "الإمارات العربية المتحدة (UAE)", "المغرب (Morocco)", "سلطنة عمان (Oman)", "الكويت (Kuwait)", "مصر (Egypt)", "الخليج العربي عموماً"];
  const niches = ["منتجات حل المشاكل اليومية", "مستحضرات التجميل والعناية بالبشرة", "أدوات المطبخ والمنزل الذكية", "اكسسوارات وعناية السيارات", "منتجات الصحة والراحة", "منتجات الأطفال والألعاب الذكية"];

  const scanMarket = async () => {
    setLoading(true);
    setError("");
    setResults([]);

    const keyToUse = apiKey || ""; 
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${keyToUse}`;

    // دمج المتغير الجديد (productCount) داخل البرومت
    const promptText = `أنت خوارزمية ذكية متخصصة في تحليل التجارة الإلكترونية وتعمل كأداة تجسس (Spy Tool) مشابهة لـ SellTheTrend.
قم بتحليل السوق في "${market}" وتحديداً في نيتش "${niche}".
استخرج ${productCount} منتجات فعلية، محددة جداً، ورابحة (Winning Products) حالياً في هذا السوق بنظام الدفع عند الاستلام (COD).

يجب أن تقوم بمحاكاة وتقدير الأرقام بذكاء كما تفعل أدوات التجسس الكبرى.

تأكد من الرد بصيغة JSON Array فقط، وتأكد من ملء جميع الحقول التالية لكل منتج:
- product_name: اسم المنتج
- category: الفئة
- image_keyword: كلمات انجليزية بسيطة للبحث عن الصورة
- why_winning: سبب النجاح
- target_audience: الجمهور
- cost_price: تكلفة الشراء (رقم)
- selling_price: سعر البيع (رقم)
- profit_margin: هامش الربح (رقم)
- saturation: درجة تشبع السوق (منخفض، متوسط، عالي)
- engagement_likes: عدد الإعجابات التقريبي للإعلانات (مثال: 15K, 2.5K)
- engagement_comments: عدد التعليقات (مثال: 1.2K, 500)
- fb_search_query: كلمة مفتاحية دقيقة بالإنجليزية للبحث عنها في مكتبة إعلانات فيسبوك (Facebook Ad Library)
- aliexpress_query: كلمة مفتاحية دقيقة بالإنجليزية للبحث عنها في علي إكسبريس`;

    const payload = {
      contents: [{ parts: [{ text: promptText }] }],
      systemInstruction: { parts: [{ text: "رد فقط بمصفوفة JSON (Array). لا تكتب أي نصوص أخرى خارج المصفوفة." }] },
      generationConfig: {
        responseMimeType: "application/json",
        responseSchema: {
          type: "ARRAY",
          items: {
            type: "OBJECT",
            properties: {
              product_name: { type: "STRING" },
              category: { type: "STRING" },
              image_keyword: { type: "STRING" },
              why_winning: { type: "STRING" },
              target_audience: { type: "STRING" },
              cost_price: { type: "NUMBER" },
              selling_price: { type: "NUMBER" },
              profit_margin: { type: "NUMBER" },
              saturation: { type: "STRING" },
              engagement_likes: { type: "STRING" },
              engagement_comments: { type: "STRING" },
              fb_search_query: { type: "STRING" },
              aliexpress_query: { type: "STRING" }
            },
            required: ["product_name", "category", "image_keyword", "why_winning", "target_audience", "cost_price", "selling_price", "profit_margin", "saturation", "engagement_likes", "engagement_comments", "fb_search_query", "aliexpress_query"]
          }
        }
      }
    };

    let attempt = 0;
    const delays = [1000, 2000, 4000, 8000, 16000];
    let success = false;

    while (attempt < 5 && !success) {
      try {
        const res = await fetch(url, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        });

        if (!res.ok) throw new Error(`خطأ في السيرفر: ${res.status}`);
        
        const data = await res.json();
        const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
        
        if (!text) throw new Error("لم يتم استلام بيانات من الذكاء الاصطناعي");
        
        const parsedData = JSON.parse(text);
        setResults(parsedData);
        success = true;
      } catch (err) {
        console.error(`محاولة ${attempt + 1} فشلت:`, err);
        if (attempt === 4) {
          setError("عذراً، السيرفر عليه ضغط كبير حالياً. يرجى المحاولة مرة أخرى.");
        } else {
          await new Promise(resolve => setTimeout(resolve, delays[attempt]));
        }
        attempt++;
      }
    }
    setLoading(false);
  };

  return (
    <div dir="rtl" className="min-h-screen bg-slate-50 font-sans text-right pb-20">
      {/* Header */}
      <div className="bg-[#0f172a] text-white py-12 px-6 border-b border-slate-800 relative overflow-hidden">
        <div className="absolute top-0 left-0 w-full h-full opacity-5 pointer-events-none" style={{ backgroundImage: 'radial-gradient(circle at 2px 2px, white 1px, transparent 0)', backgroundSize: '32px 32px' }}></div>
        <div className="max-w-7xl mx-auto relative z-10 flex flex-col md:flex-row items-center justify-between gap-6">
          <div>
            <h1 className="text-4xl md:text-5xl font-black mb-3 tracking-tight flex items-center gap-3">
              <TrendingUp className="w-10 h-10 text-emerald-400" />
              ALI Spy Pro <span className="text-sm bg-emerald-500/20 text-emerald-400 px-3 py-1 rounded-full border border-emerald-500/30">V2.5</span>
            </h1>
            <p className="text-slate-400 text-lg font-medium max-w-2xl">
              لوحة تحكم استخباراتية متقدمة بديلة لـ SellTheTrend. حلل السوق، اكتشف المنتجات، وتجسس على المنافسين.
            </p>
          </div>
          <div className="flex gap-4">
            <div className="bg-slate-800/50 p-4 rounded-xl border border-slate-700 text-center">
              <div className="text-slate-400 text-xs font-bold mb-1">الأسواق المدعومة</div>
              <div className="text-xl font-black text-white">الوطن العربي</div>
            </div>
            <div className="bg-slate-800/50 p-4 rounded-xl border border-slate-700 text-center">
              <div className="text-slate-400 text-xs font-bold mb-1">تحديث البيانات</div>
              <div className="text-xl font-black text-emerald-400 flex items-center justify-center gap-1"><Activity className="w-4 h-4"/> Live</div>
            </div>
          </div>
        </div>
      </div>

      <div className="max-w-7xl mx-auto px-6 mt-8 flex flex-col xl:flex-row gap-8">
        
        {/* Sidebar / Settings */}
        <div className="w-full xl:w-1/4">
          <div className="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 sticky top-6">
            <h2 className="text-lg font-black text-slate-800 mb-5 flex items-center gap-2 border-b border-slate-100 pb-4">
              <Target className="w-5 h-5 text-indigo-500" />
              فلاتر البحث المعمق
            </h2>
            
            <div className="mb-5">
              <label className="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">الدولة المستهدفة</label>
              <select 
                className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl font-bold text-slate-700 outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-500/20 transition-all"
                value={market}
                onChange={(e) => setMarket(e.target.value)}
              >
                {markets.map(m => <option key={m} value={m}>{m}</option>)}
              </select>
            </div>
            
            <div className="mb-5">
              <label className="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">المجال (Niche)</label>
              <select 
                className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl font-bold text-slate-700 outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-500/20 transition-all"
                value={niche}
                onChange={(e) => setNiche(e.target.value)}
              >
                {niches.map(n => <option key={n} value={n}>{n}</option>)}
              </select>
            </div>

            {/* مؤشر اختيار عدد المنتجات */}
            <div className="mb-6 bg-slate-50 p-4 rounded-xl border border-slate-200">
              <label className="flex justify-between items-center text-xs font-bold text-slate-500 uppercase tracking-wider mb-3">
                <span>عدد المنتجات المطلوبة</span>
                <span className="bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded-md font-black">{productCount} منتجات</span>
              </label>
              <input 
                type="range" 
                min="1" 
                max="10" 
                value={productCount} 
                onChange={(e) => setProductCount(parseInt(e.target.value))}
                className="w-full accent-indigo-600 cursor-pointer"
              />
              <div className="flex justify-between text-[10px] text-slate-400 font-bold mt-1">
                <span>1</span>
                <span>10</span>
              </div>
            </div>

            <div className="mb-6 border-t border-slate-100 pt-5">
              <label className="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">Pexels API (للصور الحقيقية)</label>
              <input 
                type="password" 
                className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:border-indigo-500 focus:ring-2 focus:ring-indigo-500/20 transition-all"
                placeholder="أدخل المفتاح هنا..."
                value={pexelsKey}
                onChange={(e) => setPexelsKey(e.target.value)}
              />
            </div>

            <button 
              onClick={scanMarket}
              disabled={loading}
              className={`w-full py-4 rounded-xl font-black text-base text-white shadow-md transition-all flex justify-center items-center gap-2 ${loading ? 'bg-slate-400 cursor-not-allowed' : 'bg-indigo-600 hover:bg-indigo-700 hover:-translate-y-0.5 hover:shadow-indigo-500/30'}`}
            >
              {loading ? (
                <><Loader2 className="w-5 h-5 animate-spin" /> جاري استخراج {productCount} منتجات...</>
              ) : (
                <><Search className="w-5 h-5" /> استخراج المنتجات</>
              )}
            </button>
          </div>
        </div>

        {/* Main Content */}
        <div className="w-full xl:w-3/4">
          
          {/* Error Message */}
          {error && (
            <div className="bg-red-50 border-r-4 border-red-500 text-red-700 p-5 rounded-xl mb-6 flex items-start gap-3 font-bold shadow-sm">
              <AlertTriangle className="w-6 h-6 flex-shrink-0" />
              <p>{error}</p>
            </div>
          )}

          {/* Results Area */}
          {!loading && results.length === 0 && !error && (
            <div className="bg-white rounded-2xl border border-slate-200 border-dashed p-16 text-center flex flex-col items-center justify-center">
              <div className="w-20 h-20 bg-slate-50 rounded-full flex items-center justify-center mb-4">
                <Globe className="w-10 h-10 text-slate-300" />
              </div>
              <h3 className="text-xl font-black text-slate-700 mb-2">الرادار جاهز للعمل</h3>
              <p className="text-slate-500 max-w-md">
                اختر السوق والمجال وعدد المنتجات من القائمة الجانبية واضغط على "استخراج المنتجات" لتبدأ الأداة بتحليل السوق.
              </p>
            </div>
          )}

          {results.length > 0 && (
            <div>
              <div className="flex items-center justify-between mb-6">
                <h2 className="text-2xl font-black text-slate-800 flex items-center gap-2">
                  <span className="relative flex h-3 w-3">
                    <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                    <span className="relative inline-flex rounded-full h-3 w-3 bg-emerald-500"></span>
                  </span>
                  المنتجات المكتشفة ({results.length})
                </h2>
              </div>
              
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                {results.map((product, idx) => (
                  <ProductCard key={idx} product={product} pexelsKey={pexelsKey} />
                ))}
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
