[index.html](https://github.com/user-attachments/files/26960554/index.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>공문 자동 완성 AI 시스템</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700;900&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@400;700&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Myeongjo:wght@400;700;800&display=swap');
 
        body { font-family: 'Noto Sans KR', sans-serif; }
        .progress-transition { transition: width 0.5s ease-out; }
        .page-container { display: none; }
        .page-container.active { display: block; }
        
        /* 공문 미리보기 스타일 */
        .official-doc-preview {
            width: 210mm;
            min-height: 297mm;
            padding: 20mm;
            background: white;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
            margin: 0 auto;
            border: 1px solid #ddd;
            outline: none;
            overflow: hidden;
            position: relative;
            display: flex;
            flex-direction: column;
        }
 
        /* 편집 모드일 때 시각적 피드백 - 인쇄 시에는 제거 */
        .editable-active {
            border: 2px dashed #3b82f6 !important;
            background-color: #fff !important;
        }
 
        /* 인쇄 전용 스타일 */
        @media print {
            body * { visibility: hidden; }
            .official-doc-preview, .official-doc-preview * { visibility: visible !important; }
            .official-doc-preview {
                position: absolute;
                left: 0;
                top: 0;
                width: 210mm;
                height: 297mm;
                padding: 20mm;
                margin: 0;
                border: none !important;
                box-shadow: none !important;
                background: white;
                overflow: visible !important;
            }
            .editable-active {
                border: none !important;
            }
            header, footer, .no-print { display: none !important; }
            body { background: white; }
            
            .print-border-solid {
                border-bottom: 2px solid black !important;
                background-color: transparent !important;
                height: 0 !important;
            }
        }
 
        /* 서식 도구 바 스타일 */
        .toolbar-btn {
            @apply p-2 hover:bg-gray-100 rounded transition-colors flex items-center justify-center;
        }
        .toolbar-btn.active {
            @apply bg-blue-100 text-blue-600;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-900">
 
    <!-- 상단 네비게이션 -->
    <header class="bg-white border-b border-gray-200 px-4 py-3 flex items-center justify-between sticky top-0 z-10 no-print">
        <button onclick="prevPage()" class="p-2 hover:bg-gray-100 rounded-full transition-colors">
            <i data-lucide="arrow-left" class="w-6 h-6"></i>
        </button>
        
        <div class="flex-1 max-w-2xl mx-8">
            <div class="relative h-6 bg-gray-200 rounded-full overflow-hidden">
                <div id="progressBar" class="absolute top-0 left-0 h-full bg-blue-700 progress-transition flex items-center justify-center text-[10px] text-white font-bold" style="width: 25%">
                    25%
                </div>
            </div>
        </div>
 
        <button class="p-2 hover:bg-gray-100 rounded-full transition-colors">
            <i data-lucide="more-horizontal" class="w-6 h-6"></i>
        </button>
    </header>
 
    <!-- Page 1: 계약 종류 선택 -->
    <main id="page1" class="page-container active max-w-4xl mx-auto px-6 py-16 text-center">
        <h1 class="text-3xl md:text-4xl font-extrabold mb-12 tracking-tight text-gray-800">계약의 종류를 선택해주세요</h1>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6" id="typeSelector">
            <div onclick="selectType('material-lease')" id="type-material-lease" class="type-card group p-8 bg-white border-2 border-transparent rounded-2xl cursor-pointer transition-all duration-300 transform hover:-translate-y-1 hover:shadow-xl shadow-sm">
                <div class="flex flex-col items-center">
                    <div class="mb-6 p-4 rounded-2xl bg-blue-50 group-hover:bg-white transition-colors">
                        <i data-lucide="hard-hat" class="w-8 h-8 text-blue-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">자재임대계약</h3>
                    <p class="text-sm text-gray-500">건설 자재 및 장비 임대와 관련된 공문을 작성합니다.</p>
                </div>
            </div>
            <div onclick="selectType('sales-delivery')" id="type-sales-delivery" class="type-card group p-8 bg-white border-2 border-transparent rounded-2xl cursor-pointer transition-all duration-300 transform hover:-translate-y-1 hover:shadow-xl shadow-sm">
                <div class="flex flex-col items-center">
                    <div class="mb-6 p-4 rounded-2xl bg-blue-50 group-hover:bg-white transition-colors">
                        <i data-lucide="truck" class="w-8 h-8 text-blue-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">매매계약(납품계약)</h3>
                    <p class="text-sm text-gray-500">물품 매매 및 납품 대금 청구 등의 공문을 작성합니다.</p>
                </div>
            </div>
            <div onclick="selectType('subcontract')" id="type-subcontract" class="type-card group p-8 bg-white border-2 border-transparent rounded-2xl cursor-pointer transition-all duration-300 transform hover:-translate-y-1 hover:shadow-xl shadow-sm">
                <div class="flex flex-col items-center">
                    <div class="mb-6 p-4 rounded-2xl bg-blue-50 group-hover:bg-white transition-colors">
                        <i data-lucide="file-text" class="w-8 h-8 text-blue-500"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-3">하도급계약</h3>
                    <p class="text-sm text-gray-500">공사 하도급 및 용역 계약 관련 공문을 작성합니다.</p>
                </div>
            </div>
        </div>
        <div class="mt-16">
            <button id="p1NextBtn" disabled onclick="nextPage()" class="px-12 py-4 rounded-full text-lg font-bold bg-gray-300 text-gray-500 cursor-not-allowed transition-all duration-300">다음 단계로 이동</button>
        </div>
    </main>
 
    <!-- Page 2: 기본 정보 입력 -->
    <main id="page2" class="page-container max-w-3xl mx-auto px-6 py-12 pb-32">
        <h2 class="text-2xl font-bold mb-10 text-gray-800 border-l-4 border-blue-600 pl-4">기본 정보를 입력해주세요</h2>
        <div class="space-y-10">
            <section>
                <label class="block text-sm font-semibold text-gray-700 mb-4">1. 발신인을 선택해주세요</label>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
                    <div onclick="setFormData('q1_ans', '금강공업 주식회사')" class="sender-opt p-4 border-2 rounded-xl text-center cursor-pointer transition-all border-gray-200 bg-white">금강공업 주식회사</div>
                    <div onclick="setFormData('q1_ans', '금강티엘 주식회사')" class="sender-opt p-4 border-2 rounded-xl text-center cursor-pointer transition-all border-gray-200 bg-white">금강티엘 주식회사</div>
                    <div onclick="setFormData('q1_ans', '중원엔지니어링 주식회사')" class="sender-opt p-4 border-2 rounded-xl text-center cursor-pointer transition-all border-gray-200 bg-white">중원엔지니어링 주식회사</div>
                </div>
            </section>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <section>
                    <label class="block text-sm font-semibold text-gray-700 mb-2">2. 담당자 성함 및 직책</label>
                    <input type="text" oninput="setFormData('q2_ans', this.value)" placeholder="ex. 홍길동 부장" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                </section>
                <section>
                    <label class="block text-sm font-semibold text-gray-700 mb-2">3. 담당자 연락처</label>
                    <input type="text" oninput="setFormData('q3_ans', this.value)" placeholder="ex. 02-123-4567" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
                </section>
            </div>
            <section>
                <label class="block text-sm font-semibold text-gray-700 mb-2">4. 수신처</label>
                <input type="text" oninput="setFormData('q4_ans', this.value)" placeholder="대표이사 까지 기재 ex. 대한건설 주식회사 대표이사" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <label class="block text-sm font-semibold text-gray-700 mb-2">5. 수신처 주소</label>
                <input type="text" oninput="setFormData('q5_ans', this.value)" placeholder="도로명 주소 기재" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <div class="flex justify-between items-end mb-2">
                    <label class="block text-sm font-semibold text-gray-700">6. 참조자</label>
                    <div onclick="toggleReference()" class="flex items-center gap-1 cursor-pointer select-none">
                        <i id="refIcon" data-lucide="square" class="w-4 h-4 text-gray-400"></i>
                        <span class="text-xs text-gray-500">없는 경우 체크</span>
                    </div>
                </div>
                <input id="refInput" type="text" oninput="setFormData('q6_ans', this.value)" placeholder="참조자 입력" class="w-full p-3 border border-gray-300 rounded-lg outline-none">
            </section>
            <section>
                <label class="block text-sm font-semibold text-gray-700 mb-2">7. 문서 번호</label>
                <input type="text" oninput="setFormData('q7_ans', this.value)" placeholder="ex. KH-법무-24-00000" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
        </div>
        <div class="mt-16 flex justify-center gap-4">
            <button onclick="prevPage()" class="px-10 py-3 border border-gray-300 rounded-full font-bold text-gray-600 hover:bg-gray-100">이전</button>
            <button onclick="nextPage()" class="px-10 py-3 bg-blue-600 text-white rounded-full font-bold hover:bg-blue-700 shadow-lg">다음 단계</button>
        </div>
    </main>
 
    <!-- Page 3: 세부 내용 입력 -->
    <main id="page3" class="page-container max-w-3xl mx-auto px-6 py-12 pb-32">
        <h2 class="text-2xl font-bold mb-2 text-gray-800 border-l-4 border-blue-600 pl-4">공문 세부 내용을 입력해주세요</h2>
        <div class="space-y-8 mt-10">
            <section>
                <label class="flex items-center gap-2 text-sm font-semibold text-gray-700 mb-2"><i data-lucide="calendar" class="w-4 h-4 text-blue-600"></i>8. 시행일자</label>
                <input type="date" onchange="setFormData('q8_ans', this.value)" class="w-full md:w-1/2 p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <label class="flex items-center gap-2 text-sm font-semibold text-gray-700 mb-2"><i data-lucide="calendar" class="w-4 h-4 text-blue-600"></i>9. 계약 체결일자</label>
                <input type="date" onchange="setFormData('q9_ans', this.value)" class="w-full md:w-1/2 p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <label class="flex items-center gap-2 text-sm font-semibold text-gray-700 mb-2"><i data-lucide="hard-hat" class="w-4 h-4 text-blue-600"></i>10. 현장명</label>
                <input type="text" oninput="setFormData('q10_ans', this.value)" placeholder="현장명 입력" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <label class="flex items-center gap-2 text-sm font-semibold text-gray-700 mb-2"><i data-lucide="package" class="w-4 h-4 text-blue-600"></i>11. 자재 품목</label>
                <input type="text" oninput="setFormData('q11_ans', this.value)" placeholder="ex. 갱폼, 알루미늄 거푸집" class="w-full p-3 border border-gray-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500">
            </section>
            <section>
                <label class="flex items-center gap-2 text-sm font-semibold text-gray-700 mb-2"><i data-lucide="coins" class="w-4 h-4 text-blue-600"></i>13. 미지급 금액</label>
                <div class="relative">
                    <input type="text" oninput="handleAmount(this.value)" placeholder="숫자만 입력" class="w-full p-3 pr-10 border border-gray-300 rounded-lg font-mono text-lg outline-none focus:ring-2 focus:ring-blue-500">
                    <span class="absolute right-4 top-1/2 -translate-y-1/2 font-bold text-gray-500">원</span>
                </div>
                <div id="amountPreview" class="hidden mt-3 p-4 bg-blue-50 rounded-lg border border-blue-100">
                    <p class="text-sm text-blue-800 flex justify-between items-center">
                        <span class="font-medium">공문 기재 표기:</span>
                        <span class="font-bold text-lg">금 <span id="korAmount">0</span>원</span>
                    </p>
                </div>
            </section>
        </div>
        <div class="mt-16 flex justify-center gap-4">
            <button onclick="prevPage()" class="px-10 py-3 border border-gray-300 rounded-full font-bold text-gray-600 hover:bg-gray-100">이전</button>
            <button onclick="finishAndPreview()" class="px-10 py-3 bg-blue-600 text-white rounded-full font-bold hover:bg-blue-700 shadow-lg transition-all">미리보기 및 생성</button>
        </div>
    </main>
 
    <!-- Page 4: 최종 미리보기 및 편집 -->
    <main id="page4" class="page-container pb-20 bg-slate-100 min-h-screen">
        <div class="max-w-[1000px] mx-auto px-6 py-10">
            <div class="flex flex-col md:flex-row items-start md:items-center justify-between mb-8 gap-4 no-print">
                <div>
                    <h2 class="text-2xl font-bold text-gray-800 flex items-center gap-2">
                        <i data-lucide="check-circle-2" class="w-6 h-6 text-green-600"></i>
                        공문 생성이 완료되었습니다
                    </h2>
                    <p class="text-sm text-gray-500 mt-1">서식을 자유롭게 편집한 후 인쇄하세요.</p>
                </div>
                <!-- ① Word 저장 버튼 추가 -->
                <div class="flex gap-2">
                    <button onclick="saveAsWord()" class="flex items-center gap-2 px-5 py-2.5 bg-green-700 text-white rounded-lg font-semibold hover:bg-green-800 shadow-md transition-all">
                        <i data-lucide="file-down" class="w-4 h-4"></i> Word로 저장
                    </button>
                    <button onclick="printDoc()" class="flex items-center gap-2 px-5 py-2.5 bg-blue-700 text-white rounded-lg font-semibold hover:bg-blue-800 shadow-md transition-all">
                        <i data-lucide="printer" class="w-4 h-4"></i> 인쇄하기
                    </button>
                </div>
            </div>
 
            <!-- 서식 편집 도구 (Rich Text Toolbar) -->
            <div class="bg-white border border-gray-200 rounded-t-xl p-2 flex flex-wrap gap-2 items-center no-print">
                <select onchange="execCommandWithVal('fontName', this.value)" class="border border-gray-300 rounded px-2 py-1 text-sm outline-none focus:ring-1 focus:ring-blue-500">
                    <option value="'Noto Sans KR'">고딕 (Noto Sans)</option>
                    <option value="'Noto Serif KR'">명조 (Noto Serif)</option>
                    <option value="'Nanum Myeongjo'">나눔명조</option>
                    <option value="Arial">Arial</option>
                </select>
                
                <select onchange="execCommandWithVal('fontSize', this.value)" class="border border-gray-300 rounded px-2 py-1 text-sm outline-none focus:ring-1 focus:ring-blue-500">
                    <option value="3">보통 (12pt)</option>
                    <option value="1">매우 작게</option>
                    <option value="2">작게</option>
                    <option value="4">크게</option>
                    <option value="5">매우 크게</option>
                    <option value="6">제목1</option>
                    <option value="7">제목2</option>
                </select>

                <!-- ② 기본 줄간격을 '좁게(1.4)'로 설정 -->
                <select onchange="setLineHeight(this.value)" class="border border-gray-300 rounded px-2 py-1 text-sm outline-none focus:ring-1 focus:ring-blue-500">
                    <option value="2">줄간격: 보통</option>
                    <option value="1.4" selected>줄간격: 좁게</option>
                    <option value="1.8">줄간격: 약간 좁게</option>
                    <option value="2.4">줄간격: 약간 넓게</option>
                    <option value="3">줄간격: 넓게</option>
                </select>
 
                <div class="w-px h-6 bg-gray-200 mx-1"></div>
 
                <button onclick="execCommand('bold')" class="toolbar-btn" title="굵게">
                    <i data-lucide="bold" class="w-4 h-4"></i>
                </button>
                <button onclick="execCommand('underline')" class="toolbar-btn" title="밑줄">
                    <i data-lucide="underline" class="w-4 h-4"></i>
                </button>
                
                <div class="w-px h-6 bg-gray-200 mx-1"></div>
                
                <button onclick="execCommand('justifyLeft')" class="toolbar-btn" title="좌측 정렬">
                    <i data-lucide="align-left" class="w-4 h-4"></i>
                </button>
                <button onclick="execCommand('justifyCenter')" class="toolbar-btn" title="중앙 정렬">
                    <i data-lucide="align-center" class="w-4 h-4"></i>
                </button>
                <!-- ③ 오른쪽 정렬 버튼 추가 -->
                <button onclick="execCommand('justifyRight')" class="toolbar-btn" title="우측 정렬">
                    <i data-lucide="align-right" class="w-4 h-4"></i>
                </button>
                
                <div class="ml-auto flex items-center gap-2 px-3 text-xs text-blue-600 font-medium bg-blue-50 rounded-lg py-1">
                    <i data-lucide="info" class="w-3 h-3"></i>
                    내용을 클릭하여 직접 수정할 수 있습니다
                </div>
            </div>
 
            <!-- 공문 본문 (A4 비율) -->
            <div class="official-doc-preview text-gray-900 editable-active" id="officialDoc" contenteditable="true" spellcheck="false">
                <!-- 상단 헤더 및 본문 영역 -->
                <div class="flex-1">
                    <div class="text-center mb-6">
                        <h1 id="view-q1-header" class="font-extrabold text-[2.5rem] tracking-[0.2em] mb-2">발신인 명칭</h1>
                    </div>
                    <div class="flex justify-center mb-10">
                        <div id="view-address-box" class="border border-black px-6 py-2 text-center text-sm font-medium w-full">주소 및 연락처</div>
                    </div>
                    
                    <div class="w-full h-[2px] border-b-2 border-black mb-4 print-border-solid"></div>
                    <div class="grid grid-cols-[100px_1fr] gap-y-1 text-base mb-2">
                        <span class="font-bold">문서번호 :</span> <span id="view-q7"></span>
                        <span class="font-bold">시행일자 :</span> <span id="view-q8"></span>
                        <span class="font-bold pt-2">수&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;신 :</span> <div class="pt-2"><span id="view-q4"></span><br/><span id="view-q5"></span></div>
                        <div id="view-q6-row" class="contents">
                            <span class="font-bold pt-2">참&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;조 :</span> <span class="pt-2" id="view-q6"></span>
                        </div>
                        <span class="font-bold pt-2">발&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;신 :</span> <span class="pt-2"><span id="view-q1-footer"></span> 대표이사</span>
                    </div>
                    <div class="mt-4 mb-4">
                        <div class="flex items-center gap-4 mb-2">
                            <span class="font-bold shrink-0">제&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;목 :</span>
                            <span class="text-lg font-bold" id="view-title">제목</span>
                        </div>
                        <div class="w-full h-[2px] border-b-2 border-black print-border-solid"></div>
                    </div>
                    <!-- ② 기본 줄간격 1.4(좁게) 인라인 스타일로 설정 -->
                    <div id="docBody" class="mt-10 text-base leading-8 text-justify" style="line-height: 1.4">
                        <p class="mb-8">귀 사의 번영을 기원합니다.</p>
                        <div class="space-y-6">
                            <div class="flex"><span class="mr-2">1.</span><p class="indent-4">당사는 <span id="view-q9"></span> '<span id="view-q10"></span>' 현장과 관련하여 '<span id="view-contract-label"></span>'을 체결하고, <span id="view-q11"></span> 등 자재를 성실히 공급 및 납품하였으나 <span id="view-q8-body"></span> 기준 금 <span id="view-q13"></span>원을 지급받지 못한 상태입니다.</p></div>
                            <div class="flex"><span class="mr-2">2.</span><p class="indent-4">최근 건설경기 악화와 PF 자금 경색 등에 따른 귀사의 어려움을 당사도 십분 이해하나 당사 회사 규모에 비추어 과다한 미수금 발생으로 당사는 경영상 많은 어려움을 겪고 있는 실정입니다.</p></div>
                            <div class="flex"><span class="mr-2">3.</span><p class="indent-4">당사는 귀사 현장의 공정에 차질이 생기지 않도록 최선의 노력을 다하였음에도 미지급 금액이 발생한 것을 유감스럽게 생각하며, 이에 위 대금 금 <span id="view-q13-2"></span>원의 즉시 지급을 요청드립니다. 감사합니다.</p></div>
                        </div>
                    </div>
                    <!-- ④ text-center 클래스 → style="text-align:center" 로 변경하여 정렬 기능이 동작하도록 수정 -->
                    <div class="mt-32" style="text-align: center">
                        <h2 class="font-bold inline-block text-[2.2rem]">
                            <span id="view-q1-sign"></span> <span class="ml-4">대 표 이 사</span> <span class="ml-8 text-gray-300 font-light">(인)</span>
                        </h2>
                    </div>
                    <div class="mt-20 border-t border-gray-100 pt-4 text-sm text-gray-600">
                        ※ 기타 본건과 관련된 의문사항에 대해선 담당자 <span id="view-q2"></span> (T)<span id="view-q3"></span>)으로 문의 바랍니다.
                    </div>
                </div>
                <!-- ⑤ 저작권 문구 제거 (하단 Footer 삭제) -->
            </div>
        </div>
    </main>
    <!-- ⑤ 하단 고정 Footer 저작권 문구 제거 -->

    <div id="restoreDraftModal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black/50 px-6">
        <div class="w-full max-w-md rounded-2xl bg-white p-8 text-center shadow-2xl">
            <p class="text-xl font-bold text-gray-800">저장된 내용이 있습니다. 불러오시겠습니까?</p>
            <div class="mt-8 flex justify-center gap-3">
                <button onclick="loadSavedDraft()" class="px-6 py-3 rounded-xl bg-blue-600 text-white font-semibold hover:bg-blue-700 transition-colors">불러오기</button>
                <button onclick="startNewDraft()" class="px-6 py-3 rounded-xl border border-gray-300 text-gray-700 font-semibold hover:bg-gray-100 transition-colors">새로 작성하기</button>
            </div>
        </div>
    </div>

    <script>
        // 상태 관리
        const TEMP_SAVE_KEY = 'officialDocTempSaveV1';

        function createDefaultFormData() {
            return {
                contractType: '',
                q1_ans: '', q2_ans: '', q3_ans: '', q4_ans: '', q5_ans: '', q6_ans: '', q7_ans: '',
                q8_ans: '', q9_ans: '', q10_ans: '', q11_ans: '', q13_ans: ''
            };
        }

        let currentStep = 1;
        let formData = createDefaultFormData();
        let noReference = false;
 
        // 아이콘 초기화
        lucide.createIcons();
 
        // Rich Text 편집 함수
        function execCommand(command) {
            document.execCommand(command, false, null);
        }
 
        function execCommandWithVal(command, value) {
            document.execCommand(command, false, value);
        }
 
        // 줄간격 조절 함수
        function setLineHeight(value) {
            document.getElementById('docBody').style.lineHeight = value;
        }

        function saveTempDraft(targetStep) {
            const payload = {
                currentStep: targetStep,
                noReference,
                formData
            };
            localStorage.setItem(TEMP_SAVE_KEY, JSON.stringify(payload));
        }

        function getSavedDraft() {
            const saved = localStorage.getItem(TEMP_SAVE_KEY);
            if (!saved) return null;

            try {
                const parsed = JSON.parse(saved);
                if (!parsed || typeof parsed !== 'object' || !parsed.formData) return null;
                return parsed;
            } catch (error) {
                localStorage.removeItem(TEMP_SAVE_KEY);
                return null;
            }
        }

        function showRestoreModal() {
            document.getElementById('restoreDraftModal').classList.remove('hidden');
            document.getElementById('restoreDraftModal').classList.add('flex');
        }

        function hideRestoreModal() {
            document.getElementById('restoreDraftModal').classList.add('hidden');
            document.getElementById('restoreDraftModal').classList.remove('flex');
        }

        function resetTypeSelectionUI() {
            document.querySelectorAll('.type-card').forEach(card => {
                card.classList.remove('border-blue-500', 'bg-blue-50');
            });

            const nextBtn = document.getElementById('p1NextBtn');
            nextBtn.disabled = true;
            nextBtn.classList.remove('bg-blue-600', 'text-white');
            nextBtn.classList.add('bg-gray-300', 'text-gray-500', 'cursor-not-allowed');
        }

        function resetSenderSelectionUI() {
            document.querySelectorAll('.sender-opt').forEach(opt => {
                opt.classList.remove('border-blue-600', 'bg-blue-50', 'text-blue-700', 'font-bold');
            });
        }

        function setInputValue(selector, value) {
            const element = document.querySelector(selector);
            if (element) element.value = value || '';
        }

        function resetReferenceState() {
            noReference = false;
            const icon = document.getElementById('refIcon');
            const input = document.getElementById('refInput');

            icon.setAttribute('data-lucide', 'square');
            icon.classList.remove('text-blue-600');
            icon.classList.add('text-gray-400');
            input.disabled = false;
            input.value = '';
            input.classList.remove('bg-gray-100');
            lucide.createIcons();
        }

        function resetAllState() {
            currentStep = 1;
            formData = createDefaultFormData();

            resetTypeSelectionUI();
            resetSenderSelectionUI();
            resetReferenceState();

            document.querySelectorAll('#page2 input, #page3 input').forEach(input => {
                input.value = '';
            });

            document.getElementById('amountPreview').classList.add('hidden');
            document.getElementById('korAmount').innerText = '0';

            updatePage();
        }

        function restoreFormInputs() {
            const savedNoReference = noReference;

            resetTypeSelectionUI();
            resetSenderSelectionUI();
            resetReferenceState();

            if (formData.contractType) {
                selectType(formData.contractType);
            }

            if (formData.q1_ans) {
                setFormData('q1_ans', formData.q1_ans);
            }

            setInputValue("input[oninput=\"setFormData('q2_ans', this.value)\"]", formData.q2_ans);
            setInputValue("input[oninput=\"setFormData('q3_ans', this.value)\"]", formData.q3_ans);
            setInputValue("input[oninput=\"setFormData('q4_ans', this.value)\"]", formData.q4_ans);
            setInputValue("input[oninput=\"setFormData('q5_ans', this.value)\"]", formData.q5_ans);
            setInputValue("input[oninput=\"setFormData('q7_ans', this.value)\"]", formData.q7_ans);
            setInputValue("input[onchange=\"setFormData('q8_ans', this.value)\"]", formData.q8_ans);
            setInputValue("input[onchange=\"setFormData('q9_ans', this.value)\"]", formData.q9_ans);
            setInputValue("input[oninput=\"setFormData('q10_ans', this.value)\"]", formData.q10_ans);
            setInputValue("input[oninput=\"setFormData('q11_ans', this.value)\"]", formData.q11_ans);
            setInputValue("input[oninput=\"handleAmount(this.value)\"]", formData.q13_ans);

            if (savedNoReference) {
                toggleReference();
            } else {
                setInputValue('#refInput', formData.q6_ans);
            }

            handleAmount(formData.q13_ans);
        }

        function loadSavedDraft() {
            const saved = getSavedDraft();
            if (!saved) {
                hideRestoreModal();
                return;
            }

            formData = {
                ...createDefaultFormData(),
                ...saved.formData
            };
            noReference = !!saved.noReference;

            restoreFormInputs();
            hideRestoreModal();

            if (saved.currentStep === 4) {
                finishAndPreview(false);
                return;
            }

            currentStep = saved.currentStep || 1;
            updatePage();
        }

        function startNewDraft() {
            localStorage.removeItem(TEMP_SAVE_KEY);
            hideRestoreModal();
            resetAllState();
        }

        // ─────────────────────────────────────────────────────────────
        // ① MS Word 저장 함수
        // ─────────────────────────────────────────────────────────────
        function saveAsWord() {
            const docEl = document.getElementById('officialDoc');
            const docBodyEl = document.getElementById('docBody');
            const lineHeight = docBodyEl.style.lineHeight || '1.4';

            const cloned = docEl.cloneNode(true);
            cloned.removeAttribute('contenteditable');
            cloned.removeAttribute('spellcheck');
            cloned.classList.remove('editable-active');
            cloned.style.border = 'none';
            cloned.style.boxShadow = 'none';
            cloned.style.width = '100%';
            cloned.style.minHeight = 'auto';
            cloned.style.padding = '0';
            cloned.style.margin = '0';

            // ── Fix 1: .print-border-solid → Word에서 확실히 2pt 선으로 렌더링되는 단락
            // <hr>은 Word가 표(table)로 변환하므로, border-bottom이 적용된 <p>로 대체
            cloned.querySelectorAll('.print-border-solid').forEach(el => {
                const p = document.createElement('p');
                p.style.cssText = 'margin:2px 0;padding:0;border:none;border-bottom:2pt solid black;font-size:1pt;line-height:1pt;mso-line-height-rule:exactly;height:0pt;';
                p.innerHTML = '&#8203;';
                el.parentNode.replaceChild(p, el);
            });

            // ── CSS Grid → HTML <table> 변환
            const gridDiv = cloned.querySelector('.grid');
            if (gridDiv) {
                const table = document.createElement('table');
                table.style.cssText = 'width:100%;border-collapse:collapse;font-size:1em;margin-bottom:0.5rem;';

                const items = [];
                Array.from(gridDiv.children).forEach(child => {
                    if (child.id === 'view-q6-row') {
                        if (child.style.display === 'none') return;
                        Array.from(child.children).forEach(sub => items.push(sub.cloneNode(true)));
                    } else {
                        items.push(child.cloneNode(true));
                    }
                });

                for (let i = 0; i + 1 < items.length; i += 2) {
                    const tr = document.createElement('tr');
                    const td1 = document.createElement('td');
                    td1.style.cssText = 'width:110px;font-weight:bold;padding:2px 6px 2px 0;vertical-align:top;white-space:nowrap;';
                    td1.appendChild(items[i]);
                    const td2 = document.createElement('td');
                    td2.style.cssText = 'padding:2px 0;vertical-align:top;';
                    td2.appendChild(items[i + 1]);
                    tr.appendChild(td1);
                    tr.appendChild(td2);
                    table.appendChild(tr);
                }
                gridDiv.parentNode.replaceChild(table, gridDiv);
            }

            // ── 제목 flex → <table> 행으로 변환
            const titleContainer = cloned.querySelector('.mt-4.mb-4');
            if (titleContainer) {
                const titleFlex = titleContainer.querySelector('.flex');
                if (titleFlex) {
                    const titleChildren = Array.from(titleFlex.children);
                    const table = document.createElement('table');
                    table.style.cssText = 'width:100%;border-collapse:collapse;font-size:1em;margin-top:0.5rem;';
                    const tr = document.createElement('tr');
                    const td1 = document.createElement('td');
                    td1.style.cssText = 'width:110px;font-weight:bold;padding:2px 6px 2px 0;vertical-align:top;white-space:nowrap;';
                    if (titleChildren[0]) td1.appendChild(titleChildren[0].cloneNode(true));
                    const td2 = document.createElement('td');
                    td2.style.cssText = 'padding:2px 0;vertical-align:top;font-size:1.125em;font-weight:bold;';
                    if (titleChildren[1]) td2.appendChild(titleChildren[1].cloneNode(true));
                    tr.appendChild(td1);
                    tr.appendChild(td2);
                    table.appendChild(tr);
                    titleFlex.parentNode.replaceChild(table, titleFlex);
                }
            }

            // ── Fix 2: docBody 내 1. 2. 3. flex → 번호+내용 인라인 단락으로 병합
            // Word에서 flex 미지원으로 번호 뒤 줄바꿈 발생하는 문제 해결
            const clonedDocBody = cloned.querySelector('#docBody');
            if (clonedDocBody) {
                clonedDocBody.querySelectorAll('.flex').forEach(flexDiv => {
                    const numSpan = flexDiv.querySelector('span.mr-2');
                    const contentP = flexDiv.querySelector('p');
                    if (numSpan && contentP) {
                        const p = document.createElement('p');
                        p.style.cssText = 'margin:0 0 0.8rem 0;padding:0;text-indent:0;';
                        p.innerHTML = numSpan.textContent.trim() + ' ' + contentP.innerHTML;
                        flexDiv.parentNode.replaceChild(p, flexDiv);
                    }
                });
            }

            const wordHtml = `<!DOCTYPE html>
<html xmlns:o='urn:schemas-microsoft-com:office:office'
      xmlns:w='urn:schemas-microsoft-com:office:word'
      xmlns='http://www.w3.org/TR/REC-html40'>
<head>
<meta charset='utf-8'>
<title>공문</title>
<!--[if gte mso 9]><xml>
<w:WordDocument>
  <w:View>Normal</w:View>
  <w:Zoom>90</w:Zoom>
  <w:DoNotOptimizeForBrowser/>
</w:WordDocument>
</xml><![endif]-->
<style>
  @page { margin: 20mm; size: A4 portrait; }
  body { font-family: 'Malgun Gothic', 'Noto Sans KR', sans-serif; margin: 0; padding: 0; }
  .flex-1 { flex: 1; }
  .flex { display: flex; }
  .flex-col { flex-direction: column; }
  .items-center { align-items: center; }
  .justify-center { justify-content: center; text-align: center; }
  .contents { display: contents; }
  .inline-block { display: inline-block; }
  .w-full { width: 100%; }
  .shrink-0 { flex-shrink: 0; }
  .mt-4  { margin-top: 1rem; }
  .mt-10 { margin-top: 2.5rem; }
  .mt-20 { margin-top: 5rem; }
  .mt-32 { margin-top: 8rem; }
  .mb-2  { margin-bottom: 0.5rem; }
  .mb-4  { margin-bottom: 1rem; }
  .mb-6  { margin-bottom: 1.5rem; }
  .mb-8  { margin-bottom: 2rem; }
  .mb-10 { margin-bottom: 2.5rem; }
  .mr-2  { margin-right: 0.5rem; }
  .ml-4  { margin-left: 1rem; }
  .ml-8  { margin-left: 2rem; }
  .pt-2  { padding-top: 1px; }
  .pt-4  { padding-top: 1rem; }
  .px-6  { padding-left: 1.5rem; padding-right: 1.5rem; }
  .py-2  { padding-top: 0.5rem; padding-bottom: 0.5rem; }
  .text-center  { text-align: center; }
  .text-justify { text-align: justify; }
  .text-sm   { font-size: 0.875em; }
  .text-base { font-size: 1em; }
  .text-lg   { font-size: 1.125em; }
  .font-bold      { font-weight: 700; }
  .font-extrabold { font-weight: 900; }
  .font-light     { font-weight: 300; }
  .text-gray-300  { color: #d1d5db; }
  .text-gray-400  { color: #9ca3af; }
  .text-gray-600  { color: #4b5563; }
  .tracking-\\[0\\.2em\\] { letter-spacing: 0.2em; }
  .indent-4 { text-indent: 1rem; }
  .leading-8 { line-height: ${lineHeight}; }
  #docBody   { line-height: ${lineHeight}; }
  .border       { border: 1px solid black; }
  .border-black { border-color: black; }
  .border-b-2   { border-bottom: 2px solid black; }
  .border-t     { border-top: 1px solid; }
  .border-gray-100 { border-color: #f3f4f6; }
  .h-\\[2px\\]  { height: 2px; }
  .gap-4        { gap: 1rem; }
  .space-y-6 > * + * { margin-top: 1.5rem; }
  .text-\\[2\\.5rem\\] { font-size: 2.5rem; }
  .text-\\[2\\.2rem\\] { font-size: 2.2rem; }

  /* 주소 박스: 4면 테두리 동일, 내용 크기에 맞게 */
  #view-address-box {
    display: inline-block !important;
    width: auto !important;
    text-align: center;
    border: 1px solid black !important;
    padding: 3px 20px;
    line-height: 1.4;
    vertical-align: middle;
    box-sizing: border-box;
  }
</style>
</head>
<body>
${cloned.outerHTML}
</body>
</html>`;

            const blob = new Blob(['\ufeff', wordHtml], { type: 'application/msword' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = '공문.doc';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }
 
        // ─────────────────────────────────────────────────────────────
        // 서식 깨짐 방지: Enter / Delete / Backspace 키 처리
        // ─────────────────────────────────────────────────────────────
        document.addEventListener('DOMContentLoaded', function () {
            const doc = document.getElementById('officialDoc');
 
            doc.addEventListener('keydown', function (e) {
                const sel = window.getSelection();
                if (!sel || !sel.rangeCount) return;
                const range = sel.getRangeAt(0);
 
                if (e.key === 'Enter') {
                    e.preventDefault();
                    range.deleteContents();
                    const br = document.createElement('br');
                    range.insertNode(br);
                    const afterRange = document.createRange();
                    afterRange.setStartAfter(br);
                    afterRange.collapse(true);
                    sel.removeAllRanges();
                    sel.addRange(afterRange);
                    return;
                }
 
                if (e.key === 'Delete' || e.key === 'Backspace') {
                    if (!range.collapsed) return;
                    const isDelete = e.key === 'Delete';
                    const container = range.startContainer;
                    const offset = range.startOffset;
                    const atEnd = container.nodeType === Node.TEXT_NODE
                        ? offset === container.length
                        : offset === container.childNodes.length;
                    const atStart = container.nodeType === Node.TEXT_NODE
                        ? offset === 0
                        : offset === 0;
                    if (isDelete && atEnd) {
                        let node = container.nodeType === Node.TEXT_NODE
                            ? container.parentNode : container;
                        while (node && node !== doc) {
                            if (node.nextSibling) {
                                if (node.nextSibling.nodeType === Node.ELEMENT_NODE
                                    && node.nextSibling.tagName === 'DIV') {
                                    e.preventDefault();
                                }
                                break;
                            }
                            node = node.parentNode;
                        }
                    }
                    if (!isDelete && atStart) {
                        let node = container.nodeType === Node.TEXT_NODE
                            ? container.parentNode : container;
                        while (node && node !== doc) {
                            if (node.previousSibling) {
                                if (node.previousSibling.nodeType === Node.ELEMENT_NODE
                                    && node.previousSibling.tagName === 'DIV') {
                                    e.preventDefault();
                                }
                                break;
                            }
                            node = node.parentNode;
                        }
                    }
                }
            });

            if (getSavedDraft()) {
                showRestoreModal();
            }
        });
 
        // 페이지 이동 함수
        function updatePage() {
            document.querySelectorAll('.page-container').forEach(p => p.classList.remove('active'));
            document.getElementById(`page${currentStep}`).classList.add('active');
            const progress = currentStep * 25;
            const bar = document.getElementById('progressBar');
            bar.style.width = `${progress}%`;
            bar.innerText = currentStep === 4 ? '생성 완료' : `${progress}%`;
            if(currentStep === 4) {
                bar.classList.replace('bg-blue-700', 'bg-green-600');
            } else {
                bar.classList.replace('bg-green-600', 'bg-blue-700');
            }
            window.scrollTo(0, 0);
        }
 
        function nextPage() {
            if (currentStep === 2) {
                saveTempDraft(3);
            }
            if (currentStep < 4) { currentStep++; updatePage(); }
        }
        function prevPage() {
            if (currentStep > 1) { currentStep--; updatePage(); }
        }
 
        function selectType(type) {
            formData.contractType = type;
            document.querySelectorAll('.type-card').forEach(c => {
                c.classList.remove('border-blue-500', 'bg-blue-50');
            });
            document.getElementById(`type-${type}`).classList.add('border-blue-500', 'bg-blue-50');
            const nextBtn = document.getElementById('p1NextBtn');
            nextBtn.disabled = false;
            nextBtn.classList.replace('bg-gray-300', 'bg-blue-600');
            nextBtn.classList.replace('text-gray-500', 'text-white');
            nextBtn.classList.remove('cursor-not-allowed');
        }
 
        function setFormData(key, value) {
            formData[key] = value;
            if(key === 'q1_ans') {
                document.querySelectorAll('.sender-opt').forEach(opt => {
                    if(opt.innerText === value) {
                        opt.classList.add('border-blue-600', 'bg-blue-50', 'text-blue-700', 'font-bold');
                    } else {
                        opt.classList.remove('border-blue-600', 'bg-blue-50', 'text-blue-700', 'font-bold');
                    }
                });
            }
        }
 
        function toggleReference() {
            noReference = !noReference;
            const icon = document.getElementById('refIcon');
            const input = document.getElementById('refInput');
            if(noReference) {
                icon.setAttribute('data-lucide', 'check-square');
                icon.classList.replace('text-gray-400', 'text-blue-600');
                input.disabled = true;
                input.value = "";
                input.classList.add('bg-gray-100');
                formData.q6_ans = "";
            } else {
                icon.setAttribute('data-lucide', 'square');
                icon.classList.replace('text-blue-600', 'text-gray-400');
                input.disabled = false;
                input.classList.remove('bg-gray-100');
            }
            lucide.createIcons();
        }
 
        function handleAmount(val) {
            const num = val.replace(/[^0-9]/g, '');
            formData.q13_ans = num;
            const preview = document.getElementById('amountPreview');
            const amountInput = document.querySelector("input[oninput=\"handleAmount(this.value)\"]");
            if (amountInput && amountInput.value !== num) {
                amountInput.value = num;
            }
            if(num) {
                preview.classList.remove('hidden');
                document.getElementById('korAmount').innerText = new Intl.NumberFormat('ko-KR').format(num);
            } else {
                preview.classList.add('hidden');
                document.getElementById('korAmount').innerText = '0';
            }
        }
 
        function finishAndPreview(shouldSave = true) {
            const baseAddr = "13840/경기 과천시 과천대로7다길 60";
            let addr = "";
            if (formData.q1_ans === '금강공업 주식회사') addr = `${baseAddr}(갈현동, 금강공업빌딩) / T)${formData.q3_ans}`;
            else if (formData.q1_ans === '금강티엘 주식회사') addr = `${baseAddr} 금강공업빌딩 8층 / T)${formData.q3_ans}`;
            else if (formData.q1_ans === '중원엔지니어링 주식회사') addr = `${baseAddr} 금강공업빌딩 9층 / T)${formData.q3_ans}`;
            let titleText = "자재임대료 지급 요청의 건";
            let label = "자재 임대 계약";
            if(formData.contractType === 'sales-delivery') { titleText = "자재대금 지급 요청의 건"; label = "자재 납품 계약"; }
            else if(formData.contractType === 'subcontract') { titleText = "하도급대금 지급 요청의 건"; label = "하도급 계약"; }
            document.getElementById('view-q1-header').innerText = formData.q1_ans;
            document.getElementById('view-q1-footer').innerText = formData.q1_ans;
            document.getElementById('view-q1-sign').innerText = formData.q1_ans;
            document.getElementById('view-address-box').innerText = addr;
            document.getElementById('view-q7').innerText = formData.q7_ans;
            document.getElementById('view-q8').innerText = formData.q8_ans;
            document.getElementById('view-q8-body').innerText = formData.q8_ans;
            document.getElementById('view-q4').innerText = formData.q4_ans;
            document.getElementById('view-q5').innerText = formData.q5_ans;
            if(noReference || !formData.q6_ans) document.getElementById('view-q6-row').style.display = 'none';
            else {
                document.getElementById('view-q6-row').style.display = 'contents';
                document.getElementById('view-q6').innerText = formData.q6_ans;
            }
            document.getElementById('view-title').innerText = titleText;
            document.getElementById('view-q9').innerText = formData.q9_ans;
            document.getElementById('view-q10').innerText = formData.q10_ans;
            document.getElementById('view-contract-label').innerText = label;
            document.getElementById('view-q11').innerText = formData.q11_ans;
            const formattedAmt = new Intl.NumberFormat('ko-KR').format(formData.q13_ans);
            document.getElementById('view-q13').innerText = formattedAmt;
            document.getElementById('view-q13-2').innerText = formattedAmt;
            document.getElementById('view-q2').innerText = formData.q2_ans;
            document.getElementById('view-q3').innerText = formData.q3_ans;
            currentStep = 4;
            if (shouldSave) {
                saveTempDraft(4);
            }
            updatePage();
        }
 
        function printDoc() {
            window.print();
        }
    </script>
</body>
</html>
