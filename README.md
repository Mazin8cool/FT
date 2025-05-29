# FT
statistics for forgin trade
import streamlit as st
import pandas as pd
import openai
import os

# ✓ 1. تحميل البيانات (من Excel أو قاعدة بيانات)
@st.cache_data
def load_data():
    return pd.read_excel("/mnt/data/نموذج_بيانات_تجارة_خارجية.xlsx")

# ✓ 2. إعداد واجهة المستخدم
st.set_page_config(page_title="منصة التحليل الذكي للتجارة الخارجية", layout="wide")
st.title("🧠 منصة الذكاء التوليدي - قسم التجارة الخارجية")

# ✓ 3. إدخال نصي لتحليل ذكي
user_query = st.text_area("🔹 اكتب سؤالك التحليلي:", "كيف تطورت صادرات المنتجات الغذائية في آخر 5 سنوات؟")

# ✓ 4. تحميل البيانات
with st.spinner("جاري تحميل البيانات..."):
    df = load_data()
    st.success("تم تحميل البيانات بنجاح")

st.subheader(":bar_chart: نظرة سريعة على البيانات")
st.dataframe(df.head(10))

# ✓ 5. توليد تحليل ذكي عبر GPT (تجريبي)
st.subheader(":brain: التحليل الذكي")
openai.api_key = st.secrets["OPENAI_API_KEY"] if "OPENAI_API_KEY" in st.secrets else os.getenv("OPENAI_API_KEY")

if st.button("🔄 توليد التحليل"):
    if openai.api_key:
        prompt = f"""
        لديك جدول بيانات عن التجارة الخارجية. ها هي رؤوس الأعمدة:
        {', '.join(df.columns)}
        سؤالي التحليلي: {user_query}
        استخرج رؤى تحليلية مختصرة بصيغة تقرير إحصائي باللغة العربية.
        """
        
        with st.spinner("تم إرسال الطلب للذكاء الاصطناعي..."):
            try:
                response = openai.ChatCompletion.create(
                    model="gpt-4",
                    messages=[
                        {"role": "system", "content": "أنت خبير في التحليل الإحصائي للتجارة الخارجية."},
                        {"role": "user", "content": prompt},
                    ],
                    temperature=0.4,
                    max_tokens=500
                )
                analysis = response["choices"][0]["message"]["content"]
                st.success("تم التحليل")
                st.write(analysis)
            except Exception as e:
                st.error(f"حدث خطأ: {e}")
    else:
        st.warning("رجاء تعريف OpenAI API Key في متغيرات البيئة أو في Streamlit secrets")

# ✓ 6. ميزة تصدير التقرير
st.download_button(" 📤 تصدير التحليل إلى TXT", data=analysis if 'analysis' in locals() else "", file_name="analysis_report.txt")
