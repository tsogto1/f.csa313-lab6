# Лаборатори №6: Category-Partition ба PICT

Оюутны нэр: Х.Цогтбаяр
Оюутны код: B232270072

## Хэсэг A
### A0. Олон алдааны давуу эрэмбэ
Давуу эрэмбэ: from → to → amount, өөрөөр хэлбэл NO_FROM, FROZEN, NO_TO, SAME_ACCOUNT, BAD_AMOUNT, INSUFFICIENT, LIMIT. Үндэслэл: дүрмүүд тодорхойлолтод энэ дарааллаар бичигдсэн бөгөөд from данс байхгүй бол бусад шалгалт утгагүй тул эхэлж шалгана; amount-ийн шалгалтад хамгийн түрүүнд дүнгийн хүчинтэй эсэх (≤ 0), дараа нь үлдэгдэл, хамгийн сүүлд өдрийн лимит шалгагдана.

### A1. Сонголт ба төлөөлөх утгууд
Тестийн суурь нөхцөл: from дансны үлдэгдэл 1000, өдрийн лимит 5000, ханш 1 USD = 3000 (данс MNT-ээр).

| Параметр | Сонголт | Төлөөлөх утгууд |
|---|---|---|
| from | дансны байдал | Valid, Frozen, Missing |
| to | дансны байдал | Valid, Missing, SameAsFrom |
| amount | дүн | Normal (100), ExactBalance (1000), Zero (0), OverBalance (1001), OverLimit |
| currency | валют | Same, Different |
| (далд) өдрийн лимит | өнөөдөр хэр зарцуулсан | 0 (Normal, ExactBalance үед), 4950 (OverLimit үед: 4950 + 100 > 5000) |
| (далд) үлдэгдэл | дүнтэй харьцуулалт | amount-ийн утгаар тодорхойлогдоно |

### A2. Хязгаарлалтууд
| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from=Frozen | [ERROR] | Царцаагдсан данснаас шилжүүлэхийг ERROR_FROZEN-ээр татгалзана. |
| from=Missing | [ERROR] | Байхгүй данс ERROR_NO_FROM өгнө; amount-ийн шалгалт хүрэхгүй тул amount=Normal-той л хосолно. |
| to=Missing | [ERROR] | ERROR_NO_TO; амжилттай шилжүүлэг биш тул amount=Normal-той л хосолно. |
| to=SameAsFrom | [ERROR] [IF from≠Missing] | ERROR_SAME_ACCOUNT; "from-той ижил" нь from байхгүй үед утгагүй. |
| amount=Zero | [ERROR] | ERROR_BAD_AMOUNT (amount ≤ 0). |
| amount=OverBalance | [ERROR] | ERROR_INSUFFICIENT. |
| amount=OverLimit | [ERROR] [IF далд лимит=4950] | ERROR_LIMIT; үлдэгдэлд багтсан ч лимитийг хэтрүүлэх тул далд нөхцөл (өдрийн зарцуулалт) шаардана. |
| amount=ExactBalance | [SINGLE] | Үлдэгдэлтэй яг тэнцүү хязгаарын утга амжилттай болох ёстой; нэг удаа шалгахад хангалттай. |

Тооцоо: бүрэн олонлог = 3 × 3 × 5 × 2 = 90. Хязгаарлалтын дараа: алдаа бүр нэг л удаа (7: Frozen, from-Missing, to-Missing, SameAsFrom, Zero, OverBalance, OverLimit), SINGLE нэг удаа (1), алдаагүй утгуудыг хослуулсан (Valid, Valid, Normal, currency 2 утга = 2). **90 → 10.**

### A3. Спецификацийн хүснэгт
Суурь нөхцөл: үлдэгдэл 1000, лимит 5000, лимитийн зарцуулалт 0 (өөрөөр заагаагүй бол), ханш 1 USD = 3000.

| № | from | to | amount | currency | Тайлбар | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal (100) | Same | Happy path | 201 OK |
| 2 | Valid | Valid | Normal (0.1 USD=300) | Different | Ханшийн хөрвүүлэлт | 201 OK |
| 3 | Valid | Valid | ExactBalance (1000) | Same | [SINGLE] хязгаар | 201 OK |
| 4 | Missing | Valid | Normal | Same | | ERROR_NO_FROM |
| 5 | Frozen | Valid | Normal | Same | | ERROR_FROZEN |
| 6 | Valid | Missing | Normal | Same | | ERROR_NO_TO |
| 7 | Valid | SameAsFrom | Normal | Same | | ERROR_SAME_ACCOUNT |
| 8 | Valid | Valid | Zero (0) | Same | | ERROR_BAD_AMOUNT |
| 9 | Valid | Valid | OverBalance (1001) | Same | | ERROR_INSUFFICIENT |
| 10 | Valid | Valid | OverLimit (100, зарцуулалт 4950) | Same | Далд лимит | ERROR_LIMIT |
| 11 | Valid | Valid | 1 USD (=3000) | Different | Хөрвүүлсний дараа 1000-аас их | ERROR_INSUFFICIENT |
| 12 | Frozen | Valid | Zero | Same | Давхар: from + amount | ERROR_FROZEN (from түрүүлнэ) |
| 13 | Valid | Missing | OverBalance | Same | Давхар: to + amount | ERROR_NO_TO |
| 14 | Valid | SameAsFrom | Zero | Same | Давхар: same + amount | ERROR_SAME_ACCOUNT |
| 15 | Valid | Valid | 1200, зарцуулалт 4950 | Same | Давхар: үлдэгдэл + лимит | ERROR_INSUFFICIENT (үлдэгдэл лимитээс түрүүлнэ) |

## Хэсэг B
### B1. Лекцийн browser жишээ

Model: `browser.model` (Content, Cookies: 3 утга; Popups, Addons, Attacks, Forgeries: 2 утга). Бүрэн олонлог = 3×3×2×2×2×2 = 144.

PICT: `pict commit ab76c25 2026-09-08` (results/pict-build.txt). `/r` ашиглаагүй.

| Хувилбар | Команд | Мөрийн тоо (толгойгүй) | Файл |
|---|---|---|---|
| 2-way (pairwise) | `pict browser.model` | 10 | results/browser-2way.txt |
| 3-way | `pict browser.model /o:3` | 20 | results/browser-3way.txt |
| Бүрэн олонлог | - | 144 | - |

**2-way: яагаад 9 биш 10 мөр гарав?** Лекц дээр гараар 9 мөрөнд багтаасан бол PICT түүнээс нэг мөр илүү буюу 10 мөр өгсөн. Учир нь PICT greedy алгоритм ашигладаг: алхам бүрт одоогоор хамрагдаагүй хосыг хамгийн олон хамарсан мөрийг сонгож нэмдэг, гэхдээ ингэснээр хамгийн цөөн мөр гарна гэсэн баталгаа байхгүй. Хамгийн цөөн хэмжээтэй covering array олох нь хайлтын хувьд хүнд бодлого тул PICT хурд, үр дүнгийн чанарыг тэнцвэржүүлж "хангалттай сайн" хариу өгдөг. Гэсэн ч 10 мөр нь бүрэн олонлог 144-ийн ердөө 7% юм.

**3-way:** 20 мөр гарсан нь бүрэн олонлог 144-ийн ойролцоогоор 14% (144 → 20). Гурвалсан хослол бүрийг хамрахад хамгийн олон утгатай гурван параметрийн (3×3×2) хослол буюу доод тал нь 18 мөр хэрэгтэй тул 20 нь тэр доод хязгаарт ойрхон байна. 2-way-тэй харьцуулахад мөрийн тоо 2 дахин өссөн боловч ихэнх алдаа 2-3 параметрийн харилцан үйлчлэлээс үүсдэг тул нэмэлт хамралтыг өгнө.

## Дүгнэлт
