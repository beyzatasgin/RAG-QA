#  Langflow ile RAG Tabanlı Restoran S&S Uygulaması

Langflow kullanılarak **kod yazmadan**, görsel node'larla oluşturulmuş **Retrieval-Augmented Generation (RAG)** tabanlı restoran soru-cevap chatbot'u.

---

## İçindekiler

- [Genel Bakış](#genel-bakış)
- [Akış Mimarisi](#akış-mimarisi)
- [Kurulum](#kurulum)
- [Kullanım](#kullanım)
- [GitHub Notları](#github-notları)
- [Kaynaklar](#kaynaklar)

---

## Genel Bakış

Bu uygulama, bir restoran PDF dokümanındaki (çalışma saatleri, konum, ödeme seçenekleri, menü vb.) sık sorulan sorulara otomatik cevap verir.

**Temel özellikler:**

- 📄 PDF tabanlı bilgi kaynağı (Restoran S&S dokümanı)
- 🔍 Anlamsal arama ile ilgili içeriği otomatik bulma (RAG)
- 💬 Kullanıcıya özel konuşma geçmişi (Session ID ile)
- 🧩 Tamamen görsel, kodsuz Langflow akışı

**Kullanılan teknolojiler:** Langflow · OpenAI (GPT-4o-mini + Embeddings) · DataStax Astra DB (Vector Store)

---

## Akış Mimarisi

Akıştaki tüm adımlar Langflow'da görsel node'larla bağlanır:

| Adım | Node'lar | Açıklama |
|------|----------|----------|
| **Doküman Girişi** | Read File → Split Text | PDF okunur, 1000 karakterlik parçalara (200 overlap) bölünür. |
| **Vektörleme** | OpenAI Embeddings | Parçalar `text-embedding-3-small` ile vektörlenir. |
| **Vector Store** | Astra DB | Vektörler Astra'ya yazılır; kullanıcı sorusu burada aranır (Search Query). |
| **Kullanıcı Girişi** | Chat Input, Text (Name) | Soru ve kullanıcı adı girilir. |
| **Geçmiş** | Message History | Mesajlar saklanır; çıktı Prompt'taki `{history}` alanına bağlanır. |
| **RAG Birleştirme** | Parser → Prompt Template | Arama sonuçları metne çevrilir (`context`). `context` + `history` + `question` prompt'ta birleşir. |
| **Üretim** | OpenAI (gpt-4o-mini) | Düşük temperature (~0.18) ile cevap üretilir. |
| **Çıktı** | Chat Output | Yanıt "AI" adıyla gösterilir. |

### Prompt Şablonu

Prompt Template'te kullanılan yapı:

```
Hey, answer the users question based on the following context.
The context is: {context}
Message history: {history}
Question: {question}
```

- `{context}` → Astra DB'den gelen ilgili metin (RAG sonucu)
- `{history}` → Message History'den gelen konuşma geçmişi
- `{question}` → Chat Input'tan gelen kullanıcı sorusu

### Konuşma Geçmişi

Message History node'unda **Session ID** olarak kullanıcı adı (Text Input) kullanılır. Farklı bir isim girildiğinde yeni bir oturum başlar; her kullanıcı yalnızca kendi geçmişini görür.

---

## Kurulum

### Gereksinimler

- Python 3.10+
- OpenAI API Key ([platform.openai.com/api-keys](https://platform.openai.com/api-keys))
- DataStax Astra DB hesabı ([astra.datastax.com](https://dtsx.io/3vZk6n2))

### Adım 1: Langflow'u Kurun ve Başlatın

```bash
# Kurulum (Windows)
pip install langflow --pre --force-reinstall

# Kurulum (Mac/Linux)
pip3 install langflow --pre --force-reinstall

# Çalıştırma
langflow run
```

Tarayıcıda `http://localhost:7860` adresinden arayüz açılır.

### Adım 2: Yeni Akış Oluşturun

Langflow arayüzünde **New Project → Blank Flow** seçin.  
Bu repodaki JSON dosyasını kullanmak isterseniz **Import** butonuyla yükleyin.

### Adım 3: Astra DB Ayarları

1. [Astra DB](https://dtsx.io/3vZk6n2) üzerinden hesap oluşturun.
2. **Create Database → Serverless Vector** seçin.
3. Veritabanı adı (örn. `langflow_tutorial`), provider ve bölge belirleyin.
4. Oluşturulduktan sonra şunları not edin:
   - **Application Token**
   - **API Endpoint**
   - **Collection adı**

### Adım 4: Değişkenleri Girin

Langflow'da her aşağıdaki değeri ilgili node'a **Credential** veya **Variable** olarak girin:

- `OPENAI_API_KEY`
- `ASTRA_DB_TOKEN`
- `ASTRA_DB_ENDPOINT`
- `ASTRA_COLLECTION_NAME`

> ⚠️ Bu değerleri kaynak koduna veya repoya kesinlikle eklemeyin.

### Adım 5: PDF Dosyasını Seçin

**Read File** node'unda `Resturaunt Q&A.pdf` dosyasını seçin veya proje klasörüne koyup yolunu girin.

---

## Kullanım

1. Langflow'da **Run** ile akışı başlatın.
2. **Name** alanına kullanıcı adı yazın (örn. `ahmet`).
3. **Chat Input** alanına sorunuzu yazın.
4. Yanıt, PDF bağlamı ve konuşma geçmişi kullanılarak Chat Output'ta görünür.

**Örnek sorular:**
- "Tell me about the hours of the store."
- "Can you tell me about the specials and the menu?"
- "What payment methods do you accept?"

> 💡 İlk soru bazen gecikmeli yanıt verebilir; vector store ilk kez doldurulurken bu normaldir. İkinci sorudan itibaren RAG tam olarak çalışır.

---

## GitHub Notları

| Konu | Öneri |
|------|-------|
| **Flow JSON** | `Export` ile akışı JSON olarak indirip repoya ekleyin (örn. `rag_flow.json`). Başkaları `Import` ile aynı akışı kurabilir. |
| **PDF dosyası** | `Resturaunt Q&A.pdf` repoya eklenebilir; büyük dosyalar için `.gitignore` kullanın. |
| **Gizli bilgiler** | API key ve Astra token'larını repoya kesinlikle eklemeyin. Langflow credential sistemi kullanın. |
| **Collection adı** | Astra'daki collection adının Langflow ayarıyla tutarlı olmasına dikkat edin. |

---

## Kaynaklar

- [Langflow Dokümantasyonu](https://docs.langflow.org/)
- [Langflow GitHub](https://github.com/langflow-ai/langflow)
- [DataStax Astra DB](https://dtsx.io/3vZk6n2)
- [OpenAI API Keys](https://platform.openai.com/api-keys)

