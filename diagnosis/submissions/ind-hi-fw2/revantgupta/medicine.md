# Medicine sheet: ind-hi-fw2 (revantgupta)

## Medicine 1: C1 (Boilerplate)

1. **Disease:** C1, found in 16 of 156 labelled documents. Examples:
   - `7550aa66f80ae7c0`: Ads से है परेशान? बिना Ads खबरों के लिए इनस्टॉल करें दैनिक भास्कर ऐप at offset 0
   - `3a3f617148815241`: *Yearly subscription for just ₹299 Limited Period Offer. HURRY UP! / at offset 40
   - `eb3063eaa2ebf1f4`: Download Dainik Bhaskar App to read Latest Hindi News Today at offset 720
2. **Medicine:** Delete whole lines that are site furniture: the Dainik Bhaskar app banner, "Read the latest … on amarujala.com", subscription offers, "Updated/First published … IST" timestamps, "Posted By:", "Advertisement", "Don't Miss!", password-reset lines, an empty "ये भी पढ़ें" stub, "सामान्य सूची पर लौटें", the Flash-player notice, "follow us" lines and cookie notices. The pattern must match the entire line.

   ```python
   import re
   
   # Whole lines that are site furniture, not article text. Each pattern must match the entire (stripped) line.
   LINE_PATTERNS = [
       r"Ads से है परेशान\?.*ऐप",                                  # Dainik Bhaskar app banner
       r"Download Dainik Bhaskar App.*",
       r"Read the latest and breaking Hindi news on \S+\.?",
       r"\*?Yearly subscription for just .*",
       r"(Updated|First published):? .*\d{4}.*(IST|AM|PM)",          # timestamps
       r"Posted By: .{1,60}",
       r"Advertisement",
       r"Don't Miss!?",
       r"Recover your password\.?",
       r"A password will be e-mailed to you\.?",
       r"(इसे|ये|यह) भी पढ़ें?\s*[:：]?\s*[–-]?",                    # "also read" stub with nothing after it
       r"सामान्य सूची पर लौटें",
       r"इस कंटेंट को काम करने के लिए फ़्लैश प्लेयर प्लगइन की आवश्यक्ता है।?",
       r"देश और दुनिया की ताजातरीन खबरों के लिए हमे फॉलो करें.*",
       r"हम (डाटा संग्रह टूल्स|विज्ञापन, कॉन्टेंट सुझाव).{0,40}कुकीज़?.*",   # cookie notices
   ]
   PAT = re.compile(r"^\s*(?:" + "|".join(LINE_PATTERNS) + r")\s*$\n?", flags=re.M)
   
   def medicine(text):
       return PAT.sub("", text)
   ```

3. **Action:** fix. Only whole lines are removed; a phrase inside a sentence is left alone.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 11 of 16.
5. **Side effects:** Misses C1 that is not on its own line (#9, #85, #128, #129, #138: share buttons and follow lines glued into article sentences). A real one-line sentence that is only "Advertisement" or a dateline would be deleted.

## Medicine 2: C2 (Repeated lines)

1. **Disease:** C2, found in 6 of 156 labelled documents. Examples:
   - `3c41dd2124256f65`: विद्यालयों में स्थापित इन पावर प्लांट्स में इतनी क्षमता है क… at offset 120; विद्यालयों में स्थापित इन पावर प्लांट्स में इतनी क्षमता है क… at offset 1947
   - `394e0e4242e51bc8`: आज वह करोड़ों की मालकिन हैं। at offset 726; 12 साल की उम्र में ही सपना के पिता का निधन हो गया था। पिता क… at offset 934
   - `9ac3c8da3cf4a30f`: नाग केसर दक्षिणी भारत, पूर्व बंगाल, और पूर्वी हिमालय में अधि… at offset 87
2. **Medicine:** Split into sentences (on । ? ! and newlines); keep the first copy of each sentence of 90+ characters and delete later exact copies.

   ```python
   import re
   
   SPLIT = re.compile(r"(?<=[।?!])[ \t]+|\n")
   
   def medicine(text):
       # Keep the first copy of every long sentence (>= 90 chars); delete later exact copies.
       seen, out, pos = set(), [], 0
       for m in SPLIT.finditer(text + "\n"):
           sent = text[pos:m.start()]
           key = sent.strip()
           if len(key) >= 90 and key in seen:
               pass                                  # drop the repeat, keep its separator
           else:
               out.append(sent)
               if len(key) >= 90:
                   seen.add(key)
           out.append(text[m.start():m.end()] if m.end() <= len(text) else "")
           pos = m.end()
       return "".join(out)
   ```

3. **Action:** fix. Only exact repeats of long sentences are removed; short lines (headings, "Read more") are never touched.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 4 of 6.
5. **Side effects:** Misses repeats shorter than 90 characters (2 of 6 C2 docs). At 40-80 characters it also hit clean doc #75, where a news site repeats its lead sentence at the end. Deliberate repetition (a refrain in a poem or a list of quotes) would lose its later copies.

## Medicine 3: C4 (Two documents joined)

1. **Disease:** C4, found in 26 of 156 labelled documents. Examples:
   - `9992ed6ebadee109`: महिलाओं की पारिवारिक मजबूरियों को मुद्दा at offset 406; कागज़ का सामान: 7. खाद्य प्रसंस्करण at offset 699; 8। बाइनरी ऑप्शन ऑटो ट्रेडिंग at offset 1238; क्रेन (25 टन) एक टावर से लैस at offset 2892; बिजनेस क्लास में एक उड़ान at offset 3519; सी में टुपल की अवधारणा नहीं है at offset 4112; टान्नर की छवि अरस्तू के कलात्मक at offset 4284; पुनर्संयोजन चरण में, झिल्ली उत्तेजना at offset 6627
   - `12dc3faa84e51afe`: चिमनी। आपकी संभावनाएं कोहरे में शामिल हैं, आपको स्पष्ट होना चाहिए। at offset 1117; हाल में दिल्ली हाई कोर्ट में दायर की गई एक जनहित याचिका में… at offset 1509
   - `9779ccdb13b8b17d`: बैंकले गत आर्थिक वर्षमा बैंक तथा वित्तीय संस्थाले चुक्ता पुँजीको at offset 1160; मात्र एमएसीडी - द्विआधारी विकल्प के लिए सूचक से at offset 2542; एमएसीडी - द्विआधारी विकल्प के लिए सूचक डेमो at offset 4369; बिक्री विकल्प दोनों कोटिंग्स नमी के लिए प्रतिरोधी हैं। गैर ब… at offset 5420; ऊपर एमएसीडी - द्विआधारी विकल्प के लिए सूचक की ओर at offset 6175
2. **Medicine:** Drop binary-option / forex spinner pages: a document that uses 4 or more DISTINCT trading terms (द्विआधारी विकल्प, बाइनरी ऑप्शन, फॉरेक्स, विदेशी मुद्रा, Binomo, IQ Option, ब्रोकर, बिटकॉइन, CFD …). In this pack these are the main source of C4: unrelated paragraphs stitched around trading keywords.

   ```python
   import re
   
   # Binary-option / forex spinner pages: unrelated paragraphs stitched around trading keywords.
   # Count DISTINCT trading terms; a normal finance article uses one or two, the spam uses many.
   TERMS = re.compile(r"द्विआधारी विकल्प|बाइनरी ऑप्शन|बाइनरी विकल्प|फ़ॉरेक्स|फॉरेक्स|फोरेक्स|विदेशी मुद्रा|"
                      r"Forex|Binomo|IQ Option|Olymp|ब्रोकर|दलाल|बिटकॉइन|क्रिप्टो|ट्रेडिंग प्लेटफ़?ॉर्म|CFD|सीएफडी", re.I)
   
   def medicine(text):
       if len({m.group(0).lower() for m in TERMS.finditer(text)}) >= 4:
           return None   # drop
       return text
   ```

3. **Action:** drop. The pages are stitched from unrelated fragments, so no cut can recover one coherent text.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 4 of 26.
5. **Side effects:** Only catches the trading-spam kind of C4 (4 of 26); news-plus-horoscope or two-article joins are not detected. A genuine long finance article that names 4+ instruments or brokers would be dropped. It also drops doc #9, which I labelled fix.

## Medicine 4: C5 (Lists of keywords or links)

1. **Disease:** C5, found in 11 of 156 labelled documents. Examples:
   - `9992ed6ebadee109`: शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 595; शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 1985; शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 3041; शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 4039; शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 5852; शुरूआती ट्रेडरों के लिए IQ Option फ़ॉरेक्स ट्रेडिंग पर सर्वो… at offset 7998
   - `346f4f81c4b9de1e`: Binomo के ट्रेडिंग खाते - सर्वोत्तम और विश्वसनीय दलालों / Se… at offset 2772; अच्छी खबर यह है कि द्विआधारी विकल्प व्यापारियों को किसी स्थि… at offset 2913; AnyOption दूरभाष तथा ईमेल के माध्यम से 24/7 ग्राहक सेवा उपलब… at offset 3161
   - `9338b991e6f35969`: किडनी स्टोन की समस्या at offset 18; किडनी स्टोन की समस्या at offset 60; किडनी स्टोन की समस्या at offset 267; किडनी स्टोन की समस्या at offset 470
2. **Medicine:** Same rule as C4: drop documents with 4 or more distinct trading terms. In this pack, the keyword-stuffed pages (C5) are mostly the same forex/binary-option spinner sites.

   ```python
   import re
   
   # Binary-option / forex spinner pages: unrelated paragraphs stitched around trading keywords.
   # Count DISTINCT trading terms; a normal finance article uses one or two, the spam uses many.
   TERMS = re.compile(r"द्विआधारी विकल्प|बाइनरी ऑप्शन|बाइनरी विकल्प|फ़ॉरेक्स|फॉरेक्स|फोरेक्स|विदेशी मुद्रा|"
                      r"Forex|Binomo|IQ Option|Olymp|ब्रोकर|दलाल|बिटकॉइन|क्रिप्टो|ट्रेडिंग प्लेटफ़?ॉर्म|CFD|सीएफडी", re.I)
   
   def medicine(text):
       if len({m.group(0).lower() for m in TERMS.finditer(text)}) >= 4:
           return None   # drop
       return text
   ```

3. **Action:** drop. Keyword-stuffed pages have no clean text left once the keywords are removed.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 2 of 11.
5. **Side effects:** Catches only 2 of 11 C5 documents. Health/SEO keyword pages (e.g. #71 kidney stone, #111 stomach upset) repeat a phrase but look like normal articles to any repeat-count rule I tried: every n-gram repeat threshold also hit 1-17 clean documents, so I left them out.

## Medicine 5: D3 (Mixed language)

1. **Disease:** D3, found in 13 of 156 labelled documents. Examples:
   - `8f73e0fb991da311`: Patharchatta Benefits for Pregnancy in Hindi at offset 2038
   - `8a6bf883cb1345bc`: पेट में एसिड (acidity) बनना at offset 1252
   - `35d63edf3aca61a6`: 🔷 Shayari 🔷 / at offset 69; 💦 Hindi Shayari 💦 / at offset 228; Shayari In Hindi at offset 773
2. **Medicine:** Remove the SEO keyword tail "<English search phrase> in Hindi", with its leading dash or brackets, e.g. "पत्थरचट्टा के फायदे – Patharchatta Benefits for Pregnancy in Hindi" → "पत्थरचट्टा के फायदे".

   ```python
   import re
   
   # SEO keyword tail: an English search phrase ending in "in Hindi", often glued to a Hindi title
   # with a dash, e.g. "पत्थरचट्टा के फायदे – Patharchatta Benefits for Pregnancy in Hindi".
   PAT = re.compile(r"[ \t]*(?:[–—-][ \t]*)?\(?[A-Za-z][A-Za-z0-9 '&-]{2,60}? in Hindi\b\)?", re.I)
   
   def medicine(text):
       return PAT.sub("", text)
   ```

3. **Action:** fix. Natural Hindi-English code-mixing is not touched (DISEASES.md: label it, do not judge it); only the search-keyword tail is removed.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 4 of 13.
5. **Side effects:** Catches 4 of 13 D3 documents; the rest are natural mixing, which needs no medicine. Where the English phrase is the sentence subject ("provisional certificate meaning in hindi के बारे में", "Online Business Idea in Hindi:") it leaves a sentence without a subject.

## Medicine 6: D4 (Bad machine translation)

1. **Disease:** D4, found in 22 of 156 labelled documents. Examples:
   - `6e481baf5e01324e`: कंपनी हमेशा के लिए मनोरंजन ने घोषणा की है at offset 110; पहुंच जाएगा 2019 प्रारंभ करें su Nintendo स्विच at offset 197
   - `a4d3c071f097b6bf`: मैला त्रिपुटी at offset 1682; अधिक लानत at offset 2862
   - `c500da54ea4d9701`: कोरोनावायरस हो गई है at offset 127; हो गए है at offset 582; हो गए है at offset 793; हो गए है at offset 917
2. **Medicine:** Drop documents with fixed machine-translation tells that no Hindi writer produces: "द्विआधारी विकल्प" (binary option, word by word), a company name translated as words ("कंपनी हमेशा के लिए मनोरंजन"), "मैला त्रिपुटी", and a plural subject with singular "है" after गए/किए/दिए/लिए ("… गए है").

   ```python
   import re
   
   # Fixed machine-translation tells seen in this pack: word-for-word renderings that no Hindi writer uses.
   TELLS = re.compile(r"द्विआधारी विकल्प"            # "binary option" translated word by word
                      r"|कंपनी हमेशा के लिए मनोरंजन"     # "Forever Entertainment" (a company name) translated
                      r"|मैला त्रिपुटी"                  # "dirty triplet"
                      r"|(?:हो|गए|लिए|किए|दिए) गए है(?!ं)")  # plural subject + singular "है"
   
   def medicine(text):
       return None if TELLS.search(text) else text   # drop
   ```

3. **Action:** drop. A machine-translated page is wrong throughout, so drop rather than fix.
4. **Harm test:** proxy gold set (my own documents labelled clean after a full read; the official Hindi gold set was not available to me), n=87: changed 0, dropped 0 (0.0%) → **PASS** (limit 1%). On my labelled documents with this disease it acts on 5 of 22.
5. **Side effects:** Catches 5 of 22 D4 documents; most bad translation has no fixed phrase. The "गए है" pattern also matches a plain typo (missing anusvara) in a human-written article.
