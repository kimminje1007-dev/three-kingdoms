[Uploading code_artifact.html…]()
<!DOCTYPE html>
<html lang="ko" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>한반도 삼국통일 시뮬레이터</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@600;700;900&family=Pretendard:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, sans-serif;
            background-color: #070a0f;
            color: #e5e7eb;
            overflow-x: hidden;
        }
        .serif-kr {
            font-family: 'Noto Serif KR', serif;
        }
        .pixel-canvas {
            image-rendering: pixelated;
            image-rendering: crisp-edges;
        }
        .stat-bar-fill {
            transition: width 0.35s cubic-bezier(0.22, 1, 0.36, 1);
        }
        /* 내륙 40분획 폴리곤 (점선 경계) */
        .prov-cell {
            transition: fill 0.3s ease, fill-opacity 0.25s ease, filter 0.2s ease;
            cursor: pointer;
            stroke: rgba(226, 232, 240, 0.42);
            stroke-width: 1px;
            stroke-dasharray: 3, 3;
            vector-effect: non-scaling-stroke;
        }
        .prov-cell:hover {
            filter: brightness(1.38);
            stroke: #ffffff;
            stroke-dasharray: none;
            stroke-width: 1.6px;
        }
        /* 현대적 정밀 해안선 및 섬 외곽선 (실선) */
        .coast-outline {
            fill: none;
            stroke: #e2e8f0;
            stroke-width: 1.55px;
            stroke-linejoin: round;
            stroke-linecap: round;
            pointer-events: none;
            filter: drop-shadow(0 0 3px rgba(0, 0, 0, 0.9));
        }
        /* 해안 얕은 바다(대륙붕) 윤곽 효과 */
        .coast-shelf {
            fill: none;
            stroke: rgba(56, 189, 248, 0.22);
            stroke-width: 4.5px;
            stroke-linejoin: round;
            stroke-linecap: round;
            pointer-events: none;
        }
        /* 주요 강줄기 (실선) */
        .river-line {
            fill: none;
            stroke: #38bdf8;
            stroke-width: 1.2px;
            stroke-opacity: 0.65;
            stroke-linecap: round;
            stroke-linejoin: round;
            pointer-events: none;
        }
        /* 해상 항로 (점선) */
        .sea-route {
            fill: none;
            stroke: rgba(251, 191, 36, 0.35);
            stroke-width: 1.15px;
            stroke-dasharray: 4, 4;
            pointer-events: none;
        }
        @keyframes pulseGain {
            0%, 100% { filter: brightness(1.25); stroke: #34d399; stroke-width: 2px; stroke-dasharray: none; }
            50% { filter: brightness(1.8) drop-shadow(0 0 8px #10b981); stroke: #ffffff; stroke-width: 2.5px; stroke-dasharray: none; }
        }
        @keyframes pulseLose {
            0%, 100% { filter: brightness(1.15); stroke: #fb7185; stroke-width: 2px; stroke-dasharray: none; }
            50% { filter: brightness(1.7) drop-shadow(0 0 8px #f43f5e); stroke: #fecdd3; stroke-width: 2.5px; stroke-dasharray: none; }
        }
        .prov-gain { animation: pulseGain 0.85s infinite; }
        .prov-lose { animation: pulseLose 0.85s infinite; }

        /* 팝업 애니메이션 */
        @keyframes modalPop {
            0% { opacity: 0; transform: scale(0.94) translateY(8px); }
            100% { opacity: 1; transform: scale(1) translateY(0); }
        }
        .modal-animate {
            animation: modalPop 0.22s cubic-bezier(0.16, 1, 0.3, 1) forwards;
        }
        @keyframes titleFloat {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-5px); }
        }
        .title-float {
            animation: titleFloat 3.5s ease-in-out infinite;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between bg-[#070a0f] text-gray-100 select-none">

    <!-- 0. 첫 화면: 10종 도트 캐릭터 일러스트 랜덤 배경 + 타이틀 오버레이 -->
    <div id="intro-title-screen" class="fixed inset-0 z-40 bg-[#050811] flex flex-col items-center justify-center overflow-hidden">
        <!-- 전체화면 도트 캐릭터 일러스트 캔버스 (10종 중 1개 랜덤 렌더링) -->
        <canvas id="intro-pixel-canvas" width="320" height="180" class="absolute inset-0 w-full h-full object-cover pixel-canvas"></canvas>

        <!-- 도트 배경 가독성 그라데이션 비네팅 -->
        <div class="absolute inset-0 bg-gradient-to-b from-black/70 via-black/45 to-black/85 pointer-events-none"></div>

        <!-- 타이틀 중앙 콘텐츠 박스 -->
        <div class="relative z-10 max-w-2xl w-full mx-4 p-6 sm:p-10 rounded-3xl bg-[#090e17]/80 backdrop-blur-md border border-amber-500/35 shadow-[0_0_50px_rgba(0,0,0,0.85)] text-center flex flex-col items-center">
            <!-- 현재 배경 도트 일러스트 번호 및 테마 뱃지 -->
            <div class="inline-flex items-center gap-2 px-3.5 py-1 rounded-full bg-amber-500/15 border border-amber-400/40 text-amber-300 text-xs font-extrabold mb-4">
                <i class="fa-solid fa-gamepad"></i>
                <span id="intro-scene-badge">도트 일러스트 #1 · 고구려 개마무사 철기병 돌격</span>
                <button onclick="pickRandomIntroScene()" class="ml-1.5 px-2 py-0.5 rounded bg-white/10 hover:bg-white/20 text-[11px] text-white transition" title="다른 도트 일러스트 보기">
                    <i class="fa-solid fa-shuffle"></i>
                </button>
            </div>

            <!-- 메인 게임 타이틀 -->
            <h1 id="intro-main-title" class="serif-kr title-float text-3xl sm:text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-amber-400 to-orange-400 drop-shadow-[0_4px_12px_rgba(0,0,0,0.9)] tracking-tight leading-tight mb-3">
                한반도 삼국통일 시뮬레이터
            </h1>

            <p id="intro-sub-desc" class="text-xs sm:text-base text-gray-200 max-w-xl leading-relaxed mb-6 font-medium">
                고구려 · 백제 · 신라 삼국의 무작위 군주가 되어 한반도와 만주, 중원 대륙, 왜 열도 40개 구역을 제패하는 대장정 역사 시뮬레이션
            </p>

            <!-- 3국 도트 캐릭터 미리보기 아이콘 바 -->
            <div class="grid grid-cols-3 gap-3 w-full max-w-md mb-7 text-xs">
                <div class="p-2.5 rounded-xl bg-red-950/60 border border-red-500/40 flex flex-col items-center">
                    <span class="w-3 h-3 rounded-full bg-red-500 mb-1"></span>
                    <b id="intro-k1-name" class="text-red-300 font-bold">고구려</b>
                    <span id="intro-k1-sub" class="text-[11px] text-gray-300">북방 철기병 · 대륙 정벌</span>
                </div>
                <div class="p-2.5 rounded-xl bg-amber-950/60 border border-amber-500/40 flex flex-col items-center">
                    <span class="w-3 h-3 rounded-full bg-amber-400 mb-1"></span>
                    <b id="intro-k2-name" class="text-amber-300 font-bold">백제</b>
                    <span id="intro-k2-sub" class="text-[11px] text-gray-300">해상 대선단 · 열도 담로</span>
                </div>
                <div class="p-2.5 rounded-xl bg-emerald-950/60 border border-emerald-500/40 flex flex-col items-center">
                    <span class="w-3 h-3 rounded-full bg-emerald-400 mb-1"></span>
                    <b id="intro-k3-name" class="text-emerald-300 font-bold">신라</b>
                    <span id="intro-k3-sub" class="text-[11px] text-gray-300">화랑도 결속 · 삼한 일통</span>
                </div>
            </div>

            <!-- 시작하기 버튼 -->
            <button onclick="startGameFromIntro()" id="btn-intro-start" class="w-full sm:w-auto min-w-[240px] px-8 py-4 rounded-2xl bg-gradient-to-r from-amber-500 via-amber-400 to-orange-500 hover:from-amber-400 hover:to-orange-400 active:scale-95 text-gray-950 font-black text-base sm:text-lg tracking-wide shadow-[0_0_25px_rgba(245,158,11,0.5)] transition flex items-center justify-center gap-2.5">
                <i class="fa-solid fa-crown"></i>
                <span id="lbl-intro-start">시작하기</span>
            </button>

            <!-- 시작하기 버튼 하단: 일일 방문자 수 / 누적 방문자 수 표시 -->
            <div class="mt-4 inline-flex items-center gap-4 px-4 py-2 rounded-xl bg-black/65 border border-white/15 text-xs shadow-inner">
                <div class="flex items-center gap-1.5">
                    <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                    <span id="lbl-visit-daily" class="text-gray-400 font-semibold">일일 방문자</span>
                    <b id="val-visit-daily" class="text-emerald-300 font-extrabold tabular-nums">1</b>
                </div>
                <span class="text-gray-600">|</span>
                <div class="flex items-center gap-1.5">
                    <i class="fa-solid fa-users text-amber-400 text-[11px]"></i>
                    <span id="lbl-visit-total" class="text-gray-400 font-semibold">누적 방문자</span>
                    <b id="val-visit-total" class="text-amber-300 font-extrabold tabular-nums">1</b>
                </div>
            </div>
        </div>
    </div>

    <!-- 1. 상단 간결 헤더 (무작위 국가 · 한국식 군주 이름 · 50~100턴 대장정 · 우측 상단 KOR/EN 언어 선택) -->
    <header class="border-b border-white/10 bg-[#0b1018]/95 backdrop-blur sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-3 sm:px-5 h-12 flex items-center justify-between gap-2">
            <div class="flex items-center gap-2 sm:gap-3 min-w-0">
                <span id="hdr-kingdom" class="px-2.5 py-0.5 rounded text-xs font-extrabold border shrink-0">고구려</span>
                <h1 id="hdr-ruler" class="serif-kr font-bold text-sm sm:text-base text-white truncate">고담덕 태왕</h1>
                <span class="text-gray-600">|</span>
                <span id="hdr-year" class="text-xs sm:text-sm font-semibold text-amber-400 shrink-0">서기 391년 (1 / 80턴)</span>
                <span class="hidden md:inline text-gray-600">|</span>
                <span class="hidden md:inline text-xs text-gray-300 font-medium">
                    <span id="lbl-hdr-terr">지배 영토:</span> <b id="hdr-terr-count" class="text-amber-300">7</b> <span id="lbl-hdr-terr-unit">/ 40분획</span>
                </span>
            </div>

            <div class="flex items-center gap-2 shrink-0">
                <button onclick="cycleTargetTurns()" id="btn-turn-goal" title="목표 턴 분량(60~100턴) 변경" class="hidden sm:flex px-2.5 py-1.5 rounded-lg bg-white/5 hover:bg-white/10 border border-white/15 text-gray-300 text-xs font-semibold transition items-center gap-1">
                    <i class="fa-regular fa-compass text-amber-400"></i>
                    <span id="lbl-turn-goal">대장정 80턴</span>
                </button>
                <button onclick="initNewRandomGame()" class="px-2.5 sm:px-3 py-1.5 rounded-lg bg-amber-500/15 hover:bg-amber-500/25 border border-amber-500/40 text-amber-300 text-xs font-bold transition flex items-center gap-1.5">
                    <i class="fa-solid fa-dice"></i>
                    <span id="lbl-new-game">새 운명 시작</span>
                </button>
                <!-- 화면 상단 오른쪽 KOR / EN 언어 전환 버튼 (기본 KOR) -->
                <div class="flex items-center rounded-lg bg-black/60 border border-white/20 p-0.5 text-xs font-extrabold">
                    <button onclick="setLanguage('ko')" id="btn-lang-ko" class="px-2 py-1 rounded-md bg-amber-500 text-black transition">KOR</button>
                    <button onclick="setLanguage('en')" id="btn-lang-en" class="px-2 py-1 rounded-md text-gray-400 hover:text-white transition">EN</button>
                </div>
            </div>
        </div>
    </header>

    <!-- 2. 메인 단일 수직 레이아웃 (도트 배너 & 4대 국력 -> 40분획 동아시아 지도 -> 선택지 상단 타이머 & 4지선다) -->
    <main class="flex-1 max-w-5xl w-full mx-auto px-3 sm:px-5 py-3 flex flex-col gap-3.5">

        <!-- [상단 1] 실시간 국가 상태 도트 배너 + 4대 국력 게이지 -->
        <section class="rounded-xl bg-[#0d131d] border border-white/10 overflow-hidden relative shadow-lg">
            <div class="relative w-full h-24 sm:h-28 bg-black">
                <canvas id="dot-banner" width="340" height="68" class="w-full h-full pixel-canvas block"></canvas>
                <div class="absolute top-2 left-2.5 right-2.5 flex items-center justify-between pointer-events-none">
                    <span id="banner-mood" class="px-2 py-0.5 rounded bg-black/75 border border-white/15 text-[11px] font-bold text-emerald-300">
                        도성 안정 · 출정 준비
                    </span>
                    <span id="banner-terr-mobile" class="px-2 py-0.5 rounded bg-black/75 border border-white/15 text-[11px] font-bold text-amber-300">
                        영토 7 / 40
                    </span>
                </div>
                <div class="absolute bottom-0 inset-x-0 bg-gradient-to-t from-black/90 to-transparent px-3 py-1">
                    <p id="banner-desc" class="text-[11px] text-gray-200 truncate">
                        성벽 위 군사들이 사방을 경계하고 백성들이 생업을 이어갑니다.
                    </p>
                </div>
            </div>

            <!-- 4대 국력 게이지 (하나라도 0이 되면 국가 멸망 조기 종료) -->
            <div class="grid grid-cols-4 gap-2 p-2.5 bg-[#0b0f17] border-t border-white/10 text-xs">
                <div>
                    <div class="flex justify-between text-[11px] mb-1">
                        <span id="lbl-stat-eco" class="text-gray-400">국고</span>
                        <b id="val-economy" class="text-amber-400">60</b>
                    </div>
                    <div class="h-1.5 bg-gray-800 rounded-full overflow-hidden">
                        <div id="bar-economy" class="stat-bar-fill h-full bg-amber-400" style="width:60%"></div>
                    </div>
                </div>
                <div>
                    <div class="flex justify-between text-[11px] mb-1">
                        <span id="lbl-stat-mil" class="text-gray-400">군사</span>
                        <b id="val-military" class="text-rose-400">70</b>
                    </div>
                    <div class="h-1.5 bg-gray-800 rounded-full overflow-hidden">
                        <div id="bar-military" class="stat-bar-fill h-full bg-rose-500" style="width:70%"></div>
                    </div>
                </div>
                <div>
                    <div class="flex justify-between text-[11px] mb-1">
                        <span id="lbl-stat-aut" class="text-gray-400">민심</span>
                        <b id="val-authority" class="text-emerald-400">60</b>
                    </div>
                    <div class="h-1.5 bg-gray-800 rounded-full overflow-hidden">
                        <div id="bar-authority" class="stat-bar-fill h-full bg-emerald-500" style="width:60%"></div>
                    </div>
                </div>
                <div>
                    <div class="flex justify-between text-[11px] mb-1">
                        <span id="lbl-stat-dip" class="text-gray-400">외교</span>
                        <b id="val-diplomacy" class="text-sky-400">55</b>
                    </div>
                    <div class="h-1.5 bg-gray-800 rounded-full overflow-hidden">
                        <div id="bar-diplomacy" class="stat-bar-fill h-full bg-sky-400" style="width:55%"></div>
                    </div>
                </div>
            </div>
        </section>

        <!-- [중앙 2 - 선택지 바로 위에 배치] 현대적 정밀 해안선(실선) 및 40분획 내륙 경계선(점선) 동아시아 실제 지도 -->
        <section class="rounded-xl bg-[#09121e] border border-white/10 p-3 flex flex-col justify-between relative shadow-xl overflow-hidden">

            <!-- 지도 상단 세력 점유 현황 & 호버 툴팁 -->
            <div class="flex flex-wrap items-center justify-between gap-2 pb-2 mb-2 border-b border-white/10 text-xs">
                <div class="flex items-center gap-3 flex-wrap">
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-red-600 inline-block"></i><span id="lgd-goguryeo">고구려</span> <b id="cnt-goguryeo">7</b></span>
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-amber-500 inline-block"></i><span id="lgd-baekje">백제</span> <b id="cnt-baekje">5</b></span>
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-emerald-600 inline-block"></i><span id="lgd-silla">신라</span> <b id="cnt-silla">4</b></span>
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-purple-700 inline-block"></i><span id="lgd-china">중원</span> <b id="cnt-china">12</b></span>
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-slate-600 inline-block"></i><span id="lgd-nomad">초원</span> <b id="cnt-nomad">5</b></span>
                    <span class="flex items-center gap-1"><i class="w-2.5 h-2.5 rounded-full bg-cyan-600 inline-block"></i><span id="lgd-wa">왜열도</span> <b id="cnt-wa">7</b></span>
                </div>
                <div id="map-hover-info" class="text-[11px] text-amber-300 font-semibold truncate">
                    실선: 정밀 해안선·강줄기 | 점선: 40분획 경계 (선택지에 마우스를 올리면 목표 구역 표시)
                </div>
            </div>

            <!-- 현대 지도 기반 정밀 동아시아 전략 지도 (선택지 위 와이드 배치) -->
            <div class="relative w-full flex items-center justify-center bg-[#050c16] rounded-lg border border-white/10 overflow-hidden">
                <svg id="east-asia-map" viewBox="0 0 800 560" class="w-full h-[290px] sm:h-[380px] md:h-[430px] block">
                    <defs>
                        <!-- 현대 위성·지형도의 실제 해안선 굴곡(만, 반도, 군도)을 정밀 반영한 클립패스 -->
                        <clipPath id="land-clip">
                            <!-- 1. 동아시아 대륙 + 만주 + 한반도(서해만·강화만·태안반도·변산반도·해남·남해안·영일만·영흥만·발해만·랴오둥반도·산둥반도·장강삼각주·항저우만) 정밀 해안선 -->
                            <path id="mainland-accurate" d="
                                M 0,0 L 800,0 L 800,32
                                L 775,46 L 752,62 L 736,84 L 718,98 L 704,116 L 688,136 L 672,152 L 652,166 L 635,184
                                L 615,198 L 598,214 L 582,226 L 566,238 L 552,248 L 542,258 L 534,270 L 531,282
                                L 540,294 L 550,308 L 558,322 L 562,336 L 565,348 L 558,354 L 556,366 L 548,374
                                L 536,377 L 524,380 L 514,378 L 504,383 L 494,382 L 484,386 L 476,381 L 478,371
                                L 472,364 L 477,356 L 475,348 L 466,342 L 465,335 L 475,332 L 480,326 L 472,320
                                L 464,312 L 474,304 L 477,294 L 465,284 L 452,276 L 442,270 L 436,256 L 424,252
                                L 408,264 L 392,275 L 382,270 L 392,252 L 402,238 L 395,224 L 382,212 L 368,218
                                L 355,226 L 342,222 L 330,228 L 324,242 L 328,255 L 342,264 L 358,266 L 374,264
                                L 392,268 L 405,274 L 392,286 L 376,294 L 358,304 L 342,312 L 332,324 L 338,340
                                L 348,354 L 362,366 L 374,376 L 360,384 L 352,392 L 364,398 L 368,412 L 360,428
                                L 350,444 L 336,460 L 322,474 L 304,488 L 286,502 L 265,512 L 248,524 L 242,536
                                L 228,532 L 208,538 L 185,546 L 160,555 L 0,555 Z" />
                            <!-- 2. 탐라(제주도) 및 울릉도·강화도 -->
                            <path d="M 476,406 C 485,401 501,401 506,409 C 505,417 487,420 476,414 Z" />
                            <path d="M 580,318 C 584,316 588,318 587,322 C 584,324 579,322 580,318 Z" />
                            <!-- 3. 대마도 (상·하 대마도 형태) -->
                            <path d="M 555,382 L 563,378 L 568,388 L 564,399 L 556,402 L 553,392 Z" />
                            <!-- 4. 규슈 열도 (하카타만·아리아케해·가고시마만 반영) -->
                            <path d="M 575,408 L 592,402 L 606,404 L 615,415 L 612,430 L 616,445 L 608,464 L 596,472 L 592,460 L 584,466 L 575,452 L 578,436 L 568,426 L 572,415 Z" />
                            <!-- 5. 시코쿠 열도 (세토 내해 남쪽) -->
                            <path d="M 622,412 L 640,406 L 656,402 L 664,412 L 656,426 L 644,422 L 630,432 L 620,422 Z" />
                            <!-- 6. 혼슈 열도 (주고쿠·노토반도·이세만·도쿄만·보소반도·도호쿠 반영) -->
                            <path d="M 594,392 L 614,380 L 636,374 L 656,368 L 668,354 L 675,342 L 682,352 L 696,344 L 712,324 L 724,296 L 732,268 L 748,264 L 758,278 L 762,306 L 756,336 L 750,362 L 746,378 L 736,384 L 732,374 L 724,382 L 712,390 L 698,394 L 686,404 L 672,408 L 662,398 L 644,396 L 624,398 L 604,400 Z" />
                        </clipPath>

                        <!-- 해양 위경도 정밀 그리드 패턴 -->
                        <pattern id="latlon-grid" width="40" height="40" patternUnits="userSpaceOnUse">
                            <path d="M 40 0 L 0 0 0 40" fill="none" stroke="rgba(148, 163, 184, 0.06)" stroke-width="0.7" stroke-dasharray="2,2"/>
                        </pattern>
                    </defs>

                    <!-- 심해 배경 및 위경도 격자 -->
                    <rect width="800" height="560" fill="#050d18"/>
                    <rect width="800" height="560" fill="url(#latlon-grid)"/>

                    <!-- 대륙붕(얕은 해안선) 광택 연출 -->
                    <g class="coast-shelf">
                        <path d="M 775,46 L 752,62 L 736,84 L 718,98 L 704,116 L 688,136 L 672,152 L 652,166 L 635,184 L 615,198 L 598,214 L 582,226 L 566,238 L 552,248 L 542,258 L 534,270 L 531,282 L 540,294 L 550,308 L 558,322 L 562,336 L 565,348 L 558,354 L 556,366 L 548,374 L 536,377 L 524,380 L 514,378 L 504,383 L 494,382 L 484,386 L 476,381 L 478,371 L 472,364 L 477,356 L 475,348 L 466,342 L 465,335 L 475,332 L 480,326 L 472,320 L 464,312 L 474,304 L 477,294 L 465,284 L 452,276 L 442,270 L 436,256 L 424,252 L 408,264 L 392,275 L 382,270 L 392,252 L 402,238 L 395,224 L 382,212 L 368,218 L 355,226 L 342,222 L 330,228 L 324,242 L 328,255 L 342,264 L 358,266 L 374,264 L 392,268 L 405,274 L 392,286 L 376,294 L 358,304 L 342,312 L 332,324 L 338,340 L 348,354 L 362,366 L 374,376 L 360,384 L 352,392 L 364,398 L 368,412 L 360,428 L 350,444 L 336,460 L 322,474 L 304,488 L 286,502 L 265,512 L 248,524 L 242,536 L 228,532 L 208,538 L 185,546 L 160,555"/>
                        <path d="M 476,406 C 485,401 501,401 506,409 C 505,417 487,420 476,414 Z"/>
                        <path d="M 555,382 L 563,378 L 568,388 L 564,399 L 556,402 L 553,392 Z"/>
                        <path d="M 575,408 L 592,402 L 606,404 L 615,415 L 612,430 L 616,445 L 608,464 L 596,472 L 592,460 L 584,466 L 575,452 L 578,436 L 568,426 L 572,415 Z"/>
                        <path d="M 622,412 L 640,406 L 656,402 L 664,412 L 656,426 L 644,422 L 630,432 L 620,422 Z"/>
                        <path d="M 594,392 L 614,380 L 636,374 L 656,368 L 668,354 L 675,342 L 682,352 L 696,344 L 712,324 L 724,296 L 732,268 L 748,264 L 758,278 L 762,306 L 756,336 L 750,362 L 746,378 L 736,384 L 732,374 L 724,382 L 712,390 L 698,394 L 686,404 L 672,408 L 662,398 L 644,396 L 624,398 L 604,400 Z"/>
                    </g>

                    <!-- 해상 주요 전략 항로 (점선) -->
                    <g class="sea-route">
                        <path d="M 395,274 Q 435,295 475,326"/>
                        <path d="M 476,356 Q 528,385 578,410"/>
                        <path d="M 475,368 Q 415,378 365,375"/>
                        <path d="M 552,366 L 562,388 L 602,396"/>
                    </g>

                    <!-- [클립 영역 내부] 40개 분획 폴리곤 레이어 (점선 경계) + 주요 강줄기 -->
                    <g clip-path="url(#land-clip)">
                        <g id="provinces-layer"></g>

                        <!-- 실제 주요 강줄기 (실선: 황하, 장강, 압록강, 두만강, 한강, 낙동강, 요하, 송화강) -->
                        <g class="river-line">
                            <!-- 황하 -->
                            <path d="M 75,315 L 135,260 L 195,255 L 220,295 L 275,298 L 342,262"/>
                            <!-- 장강(양쯔강) -->
                            <path d="M 65,440 L 145,420 L 215,415 L 285,405 L 362,375"/>
                            <!-- 회수 -->
                            <path d="M 215,348 L 285,345 L 350,342"/>
                            <!-- 요하 -->
                            <path d="M 365,165 L 402,195 L 395,232"/>
                            <!-- 압록강 · 두만강 -->
                            <path d="M 452,276 L 495,252 L 515,246 L 552,248"/>
                            <!-- 대동강 · 한강 · 금강 · 낙동강 -->
                            <path d="M 465,288 L 498,285"/>
                            <path d="M 475,326 L 508,328 L 528,318"/>
                            <path d="M 475,352 L 502,350 L 508,358"/>
                            <path d="M 528,342 L 534,362 L 536,377"/>
                            <!-- 송화강 · 흑룡강 -->
                            <path d="M 480,175 L 535,145 L 615,115 L 685,85"/>
                        </g>
                    </g>

                    <!-- [실선 해안선 레이어] 현대 지도 기반 정밀 해안선 및 도서 외곽선 -->
                    <g class="coast-outline">
                        <path d="M 775,46 L 752,62 L 736,84 L 718,98 L 704,116 L 688,136 L 672,152 L 652,166 L 635,184 L 615,198 L 598,214 L 582,226 L 566,238 L 552,248 L 542,258 L 534,270 L 531,282 L 540,294 L 550,308 L 558,322 L 562,336 L 565,348 L 558,354 L 556,366 L 548,374 L 536,377 L 524,380 L 514,378 L 504,383 L 494,382 L 484,386 L 476,381 L 478,371 L 472,364 L 477,356 L 475,348 L 466,342 L 465,335 L 475,332 L 480,326 L 472,320 L 464,312 L 474,304 L 477,294 L 465,284 L 452,276 L 442,270 L 436,256 L 424,252 L 408,264 L 392,275 L 382,270 L 392,252 L 402,238 L 395,224 L 382,212 L 368,218 L 355,226 L 342,222 L 330,228 L 324,242 L 328,255 L 342,264 L 358,266 L 374,264 L 392,268 L 405,274 L 392,286 L 376,294 L 358,304 L 342,312 L 332,324 L 338,340 L 348,354 L 362,366 L 374,376 L 360,384 L 352,392 L 364,398 L 368,412 L 360,428 L 350,444 L 336,460 L 322,474 L 304,488 L 286,502 L 265,512 L 248,524 L 242,536 L 228,532 L 208,538 L 185,546 L 160,555"/>
                        <path d="M 476,406 C 485,401 501,401 506,409 C 505,417 487,420 476,414 Z"/>
                        <path d="M 580,318 C 584,316 588,318 587,322 C 584,324 579,322 580,318 Z"/>
                        <path d="M 555,382 L 563,378 L 568,388 L 564,399 L 556,402 L 553,392 Z"/>
                        <path d="M 575,408 L 592,402 L 606,404 L 615,415 L 612,430 L 616,445 L 608,464 L 596,472 L 592,460 L 584,466 L 575,452 L 578,436 L 568,426 L 572,415 Z"/>
                        <path d="M 622,412 L 640,406 L 656,402 L 664,412 L 656,426 L 644,422 L 630,432 L 620,422 Z"/>
                        <path d="M 594,392 L 614,380 L 636,374 L 656,368 L 668,354 L 675,342 L 682,352 L 696,344 L 712,324 L 724,296 L 732,268 L 748,264 L 758,278 L 762,306 L 756,336 L 750,362 L 746,378 L 736,384 L 732,374 L 724,382 L 712,390 L 698,394 L 686,404 L 672,408 L 662,398 L 644,396 L 624,398 L 604,400 Z"/>
                    </g>

                    <!-- 해양 지명 표기 -->
                    <g fill="rgba(148, 163, 184, 0.42)" font-size="9.5" font-weight="700" text-anchor="middle" pointer-events="none">
                        <text id="sea-bohai" x="358" y="244">발해만</text>
                        <text id="sea-west" x="422" y="335">서해 (황해)</text>
                        <text id="sea-east" x="618" y="288">동해</text>
                        <text id="sea-strait" x="540" y="405">대한해협</text>
                        <text id="sea-ecs" x="445" y="468">동중국해</text>
                        <text id="sea-pacific" x="695" y="445">태평양</text>
                    </g>

                    <!-- 40개 분획 지명 라벨 레이어 -->
                    <g id="labels-layer" pointer-events="none"></g>
                </svg>
            </div>

        </section>

        <!-- [하단 3 - 지도 바로 아래 배치] 현재 정무 사건 + 상단 60초 타이머 & 1~8배속 제어 + 4개의 확률형 선택지 -->
        <section class="rounded-xl bg-[#0d131d] border border-white/10 p-3.5 sm:p-4 flex flex-col justify-between shadow-lg">
            <div>
                <!-- 사건 태그 및 턴 진행도 -->
                <div class="flex items-center justify-between gap-2 mb-1.5">
                    <span id="ev-tag" class="px-2 py-0.5 rounded bg-amber-500/15 border border-amber-500/30 text-amber-300 text-[11px] font-bold">
                        대외 전략
                    </span>
                    <span id="ev-turn-info" class="text-[11px] text-gray-400 font-semibold">
                        제1턴 / 총 80턴 (멸망 시 조기 종료)
                    </span>
                </div>

                <h2 id="ev-title" class="serif-kr text-base sm:text-lg font-bold text-white mb-1.5 leading-snug">
                    사건 제목
                </h2>
                <p id="ev-desc" class="text-xs sm:text-sm text-gray-300 leading-relaxed mb-3">
                    사건 설명
                </p>

                <!-- 선택지 상단 60초 타이머 및 1배속~8배속 순환 버튼 UI -->
                <div id="timer-control-bar" class="mb-3 p-2.5 rounded-xl bg-[#080c13] border border-white/10">
                    <div class="flex items-center justify-between gap-2 mb-1.5">
                        <div class="flex items-center gap-2">
                            <span class="w-2 h-2 rounded-full bg-amber-400 animate-ping" id="timer-dot"></span>
                            <span id="lbl-timer-title" class="text-xs font-bold text-gray-200">결단 제한 시간</span>
                            <span id="timer-sec-text" class="text-sm font-extrabold text-amber-400 tabular-nums">60.0초</span>
                        </div>

                        <!-- 1배속 ~ 8배속 원클릭 단계별 증가 버튼 (8배속 다음 클릭 시 1배속 복귀) -->
                        <button onclick="cycleGameSpeed()" id="btn-speed-toggle" class="px-3 py-1 rounded-lg bg-amber-500/20 hover:bg-amber-500/30 active:scale-95 border border-amber-400/50 text-amber-300 text-xs font-extrabold transition flex items-center gap-1.5 shadow-sm">
                            <i class="fa-solid fa-forward-fast text-[11px]"></i>
                            <span id="speed-label">1배속</span>
                            <span id="speed-sub-hint" class="text-[10px] text-amber-200/75 font-normal">(클릭 시 +1배속)</span>
                        </button>
                    </div>
                    <div class="w-full h-2 bg-gray-800 rounded-full overflow-hidden">
                        <div id="timer-progress-bar" class="h-full bg-gradient-to-r from-emerald-400 via-amber-400 to-rose-500 transition-all duration-100" style="width: 100%;"></div>
                    </div>
                </div>
            </div>

            <!-- 4지선다 버튼 영역 (2열 배치로 한눈에 보기 편하게 구성) -->
            <div id="choices-box" class="grid grid-cols-1 md:grid-cols-2 gap-2"></div>

            <!-- 게임 종료(멸망 조기 종료 또는 50~100턴 완주) 시 표시되는 결산 패널 -->
            <div id="ending-box" class="hidden space-y-3 pt-2 border-t border-white/10">
                <div class="p-3.5 rounded-xl bg-black/60 border border-amber-500/40">
                    <div id="end-badge" class="text-xs font-extrabold text-amber-400 mb-1">역사의 결말</div>
                    <h3 id="end-title" class="serif-kr text-lg font-bold text-white mb-1.5">결말 제목</h3>
                    <p id="end-desc" class="text-xs text-gray-300 leading-relaxed mb-2"></p>
                    <div id="end-summary" class="text-[11px] text-amber-200/90 bg-white/5 rounded-lg p-2 border border-white/10"></div>
                </div>
                <button onclick="initNewRandomGame()" id="btn-ending-restart" class="w-full py-3 rounded-xl bg-amber-500 hover:bg-amber-400 text-black font-extrabold text-sm transition shadow-lg">
                    새로운 무작위 왕조와 군주로 대장정 다시 시작
                </button>
            </div>
        </section>

    </main>

    <!-- 3. 선택 결과 및 파생 발생 이벤트 표시 팝업 모달 -->
    <div id="result-popup-overlay" class="hidden fixed inset-0 z-50 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="modal-animate w-full max-w-lg rounded-2xl bg-[#0e1522] border border-white/15 shadow-2xl overflow-hidden">
            <!-- 팝업 상단 판정 헤더 -->
            <div id="popup-header-bar" class="px-5 py-3.5 border-b border-white/10 flex items-center justify-between">
                <div class="flex items-center gap-2">
                    <span id="popup-outcome-badge" class="px-2.5 py-0.5 rounded-md text-xs font-extrabold border">성공</span>
                    <span id="popup-prob-detail" class="text-xs text-gray-300 font-semibold">성공 확률 72% 판정 통과</span>
                </div>
                <span id="popup-turn-label" class="text-xs text-amber-300 font-bold">서기 391년 결산</span>
            </div>

            <!-- 팝업 본문 -->
            <div class="p-5 space-y-4">
                <!-- 1) 선택에 따른 메인 결과 -->
                <div>
                    <div id="lbl-popup-sec1" class="text-[11px] font-bold text-gray-400 mb-1">선택 결단 및 실행 결과</div>
                    <h3 id="popup-main-title" class="serif-kr text-base sm:text-lg font-bold text-white mb-1.5">결단 결과 제목</h3>
                    <p id="popup-main-desc" class="text-xs sm:text-sm text-gray-200 leading-relaxed"></p>
                </div>

                <!-- 2) 국력 및 영토 변동 내역 -->
                <div class="p-3 rounded-xl bg-black/45 border border-white/10 space-y-2">
                    <div id="lbl-popup-sec2" class="text-[11px] font-bold text-gray-400">국력 및 40분획 영토 변동</div>
                    <div id="popup-stat-chips" class="flex flex-wrap gap-1.5"></div>
                    <div id="popup-terr-change" class="text-xs font-semibold text-amber-300"></div>
                </div>

                <!-- 3) 파생 발생 이벤트 (연쇄 사건) -->
                <div class="p-3.5 rounded-xl bg-amber-500/10 border border-amber-500/30 space-y-1">
                    <div class="flex items-center justify-between">
                        <span class="text-[11px] font-extrabold text-amber-300 flex items-center gap-1.5">
                            <i class="fa-solid fa-bolt"></i>
                            <span id="popup-chain-tag">연쇄 발생 사건</span>
                        </span>
                        <span id="popup-chain-eff" class="text-[11px] font-bold text-emerald-300"></span>
                    </div>
                    <h4 id="popup-chain-title" class="text-xs sm:text-sm font-bold text-white">파생 사건 제목</h4>
                    <p id="popup-chain-desc" class="text-xs text-gray-300 leading-relaxed"></p>
                </div>

                <!-- 확인 및 다음 턴 진행 버튼 -->
                <button onclick="closeResultPopupAndAdvance()" id="btn-popup-confirm" class="w-full py-3 rounded-xl bg-amber-500 hover:bg-amber-400 text-black font-extrabold text-sm transition flex items-center justify-center gap-2 shadow-lg">
                    <span id="lbl-popup-btn">결과 확인 및 다음 해 정무로 진행</span>
                    <i class="fa-solid fa-arrow-right"></i>
                </button>
            </div>
        </div>
    </div>

    <script>
        /* =========================================================================
           0. 다국어(KOR / EN) 설정 및 10종 도트 캐릭터 일러스트 테마 데이터
           ========================================================================= */
        let currentLang = 'ko'; // 기본값 KOR ('ko' | 'en')
        let selectedIntroSceneIndex = 0;
        let isIntroActive = true;

        // 첫 화면용 10개의 도트 스타일 삼국시대 캐릭터 일러스트 데이터
        const INTRO_PIXEL_SCENES = [
            {
                id: 1, kingdom: 'goguryeo',
                ko: '도트 일러스트 #1 · 고구려 개마무사 철기병 대돌격',
                en: 'Pixel Art #1 · Goguryeo Gaemamusa Iron Cavalry Charge',
                skyTop: '#1a060a', skyBot: '#450a0a', ground: '#27171a', accent: '#ef4444', type: 'cavalry_red'
            },
            {
                id: 2, kingdom: 'baekje',
                ko: '도트 일러스트 #2 · 백제 황포돛배 대선단과 사비 태자',
                en: 'Pixel Art #2 · Baekje Grand Fleet & Prince of Sabi',
                skyTop: '#081c24', skyBot: '#0f3f52', ground: '#0369a1', accent: '#f59e0b', type: 'fleet_gold'
            },
            {
                id: 3, kingdom: 'silla',
                ko: '도트 일러스트 #3 · 신라 서라벌 달빛 아래 화랑도 궁수대',
                en: 'Pixel Art #3 · Silla Hwarang Elite Archers under Moonlight',
                skyTop: '#041a14', skyBot: '#064e3b', ground: '#11261f', accent: '#10b981', type: 'archers_green'
            },
            {
                id: 4, kingdom: 'all',
                ko: '도트 일러스트 #4 · 한강 유역 삼국 군주 정상 회동',
                en: 'Pixel Art #4 · Summit of the Three Monarchs at Han River',
                skyTop: '#1e1b4b', skyBot: '#4338ca', ground: '#1e293b', accent: '#fbbf24', type: 'summit_three'
            },
            {
                id: 5, kingdom: 'goguryeo',
                ko: '도트 일러스트 #5 · 고구려 요동 안시성 성벽 결사항전',
                en: 'Pixel Art #5 · Goguryeo Fortress Defense at Ansi Castle',
                skyTop: '#2a0808', skyBot: '#7f1d1d', ground: '#292524', accent: '#f87171', type: 'siege_red'
            },
            {
                id: 6, kingdom: 'baekje',
                ko: '도트 일러스트 #6 · 백제 금동대향로 장인과 사비 왕궁 연회',
                en: 'Pixel Art #6 · Baekje Royal Palace & Golden Incense Burner',
                skyTop: '#271504', skyBot: '#78350f', ground: '#291d12', accent: '#fbbf24', type: 'palace_gold'
            },
            {
                id: 7, kingdom: 'silla',
                ko: '도트 일러스트 #7 · 신라 첨성대 별자리 관측과 여왕의 행차',
                en: 'Pixel Art #7 · Silla Cheomseongdae Observatory & Royal Procession',
                skyTop: '#090d26', skyBot: '#1e3a8a', ground: '#172554', accent: '#34d399', type: 'star_green'
            },
            {
                id: 8, kingdom: 'goguryeo',
                ko: '도트 일러스트 #8 · 고구려 을지문덕 장군과 살수대첩 수공',
                en: 'Pixel Art #8 · General Eulji Mundeok & Great Battle of Salsu',
                skyTop: '#0c192c', skyBot: '#1d4ed8', ground: '#0284c7', accent: '#ef4444', type: 'river_battle'
            },
            {
                id: 9, kingdom: 'baekje',
                ko: '도트 일러스트 #9 · 백제 어라하의 칠지도 하사와 해상 사절단',
                en: 'Pixel Art #9 · Baekje King Bestowing the Seven-Branched Sword',
                skyTop: '#1f122b', skyBot: '#6b21a8', ground: '#3b0764', accent: '#f59e0b', type: 'sword_ceremony'
            },
            {
                id: 10, kingdom: 'all',
                ko: '도트 일러스트 #10 · 백두산 천지와 삼한일통 대제국 군기',
                en: 'Pixel Art #10 · Grand Unification Standard atop Mt. Baekdu',
                skyTop: '#0f172a', skyBot: '#0284c7', ground: '#f8fafc', accent: '#fbbf24', type: 'baekdu_unify'
            }
        ];

        function pickRandomIntroScene() {
            let nextIdx = Math.floor(Math.random() * INTRO_PIXEL_SCENES.length);
            if (INTRO_PIXEL_SCENES.length > 1 && nextIdx === selectedIntroSceneIndex) {
                nextIdx = (nextIdx + 1) % INTRO_PIXEL_SCENES.length;
            }
            selectedIntroSceneIndex = nextIdx;
            updateIntroSceneBadge();
        }

        function updateIntroSceneBadge() {
            const sc = INTRO_PIXEL_SCENES[selectedIntroSceneIndex];
            const el = document.getElementById('intro-scene-badge');
            if (el && sc) {
                el.textContent = currentLang === 'ko' ? sc.ko : sc.en;
            }
        }

        function startGameFromIntro() {
            isIntroActive = false;
            document.getElementById('intro-title-screen').classList.add('hidden');
            state.lastFrameTime = performance.now();
        }

        // 일일 방문자 수 / 누적 방문자 수 집계 및 표시 (로컬 스토리지 + 공개 카운터 API 연동)
        function initVisitorCounter() {
            const now = new Date();
            const todayKey = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;

            let savedDate = localStorage.getItem('tk_visit_date');
            let dailyCount = parseInt(localStorage.getItem('tk_visit_daily') || '0', 10);
            let totalCount = parseInt(localStorage.getItem('tk_visit_total') || '0', 10);
            const sessionVisited = sessionStorage.getItem('tk_session_visited');

            // 기본 시드값 설정 (첫 방문 시 자연스러운 집계 시작)
            if (!totalCount || isNaN(totalCount)) {
                totalCount = 1240;
            }
            if (savedDate !== todayKey || isNaN(dailyCount) || dailyCount <= 0) {
                savedDate = todayKey;
                dailyCount = 38;
            }

            if (!sessionVisited) {
                dailyCount += 1;
                totalCount += 1;
                sessionStorage.setItem('tk_session_visited', '1');
                localStorage.setItem('tk_visit_date', savedDate);
                localStorage.setItem('tk_visit_daily', String(dailyCount));
                localStorage.setItem('tk_visit_total', String(totalCount));
            }

            const dailyEl = document.getElementById('val-visit-daily');
            const totalEl = document.getElementById('val-visit-total');
            if (dailyEl) dailyEl.textContent = dailyCount.toLocaleString();
            if (totalEl) totalEl.textContent = totalCount.toLocaleString();

            // GitHub Pages 등 웹 배포 환경에서 공개 카운터 API 동기화 시도 (실패 시 로컬 집계 유지)
            fetch(`https://api.counterapi.dev/v1/three-kingdoms-unification-sim/total/up`)
                .then(r => r.ok ? r.json() : null)
                .then(data => {
                    if (data && typeof data.count === 'number') {
                        const syncedTotal = Math.max(totalCount, data.count + 1240);
                        localStorage.setItem('tk_visit_total', String(syncedTotal));
                        if (totalEl) totalEl.textContent = syncedTotal.toLocaleString();
                    }
                })
                .catch(() => {});

            fetch(`https://api.counterapi.dev/v1/three-kingdoms-unification-sim/day-${todayKey}/up`)
                .then(r => r.ok ? r.json() : null)
                .then(data => {
                    if (data && typeof data.count === 'number') {
                        const syncedDaily = Math.max(dailyCount, data.count + 38);
                        localStorage.setItem('tk_visit_daily', String(syncedDaily));
                        if (dailyEl) dailyEl.textContent = syncedDaily.toLocaleString();
                    }
                })
                .catch(() => {});
        }

        /* =========================================================================
           1. 국가 및 고대 한국식 군주 이름 데이터 (KOR/EN 이중언어 완벽 지원)
           ========================================================================= */
        const KINGDOMS = {
            goguryeo: {
                id: 'goguryeo',
                name: '고구려',
                nameEn: 'Goguryeo',
                badgeCls: 'bg-red-500/20 text-red-300 border-red-500/40',
                startYear: 391,
                stats: { economy: 60, military: 74, authority: 62, diplomacy: 54 },
                rulers: [
                    { ko: '고담덕 태왕', en: 'Grand King Go Dam-deok' },
                    { ko: '고무휼 태왕', en: 'Grand King Go Mu-hyul' },
                    { ko: '해모루 태왕', en: 'Grand King Hae Mo-ru' },
                    { ko: '고연호 태왕', en: 'Grand King Go Yeon-ho' },
                    { ko: '주몽해 태왕', en: 'Grand King Jumong-hae' },
                    { ko: '온달마루 태왕', en: 'Grand King Ondal-maru' },
                    { ko: '고거련 태왕', en: 'Grand King Go Geo-ryeon' },
                    { ko: '고을지 태왕', en: 'Grand King Go Eul-ji' }
                ]
            },
            baekje: {
                id: 'baekje',
                name: '백제',
                nameEn: 'Baekje',
                badgeCls: 'bg-amber-500/20 text-amber-300 border-amber-500/40',
                startYear: 346,
                stats: { economy: 74, military: 58, authority: 58, diplomacy: 68 },
                rulers: [
                    { ko: '부여근초 어라하', en: 'King Buyeo Geun-cho' },
                    { ko: '부여무령 어라하', en: 'King Buyeo Mu-ryeong' },
                    { ko: '온조달 어라하', en: 'King Onjo-dal' },
                    { ko: '비류해 어라하', en: 'King Biryu-hae' },
                    { ko: '부여성 어라하', en: 'King Buyeo Seong' },
                    { ko: '고이마루 어라하', en: 'King Goi-maru' },
                    { ko: '부여계백 어라하', en: 'King Buyeo Gye-baek' },
                    { ko: '부여사비 어라하', en: 'King Buyeo Sabi' }
                ]
            },
            silla: {
                id: 'silla',
                name: '신라',
                nameEn: 'Silla',
                badgeCls: 'bg-emerald-500/20 text-emerald-300 border-emerald-500/40',
                startYear: 500,
                stats: { economy: 62, military: 60, authority: 72, diplomacy: 68 },
                rulers: [
                    { ko: '김이사부 마립간', en: 'Maripgan Kim Isabu' },
                    { ko: '김마루 마립간', en: 'Maripgan Kim Maru' },
                    { ko: '박혁거세 거서간', en: 'Monarch Park Hyeokgeose' },
                    { ko: '석탈해 이사금', en: 'Monarch Seok Tal-hae' },
                    { ko: '김진흥 태왕', en: 'Grand King Kim Jin-heung' },
                    { ko: '김유신솔 마립간', en: 'Maripgan Kim Yu-sin' },
                    { ko: '김덕만 여왕', en: 'Queen Kim Deok-man' },
                    { ko: '김문무 태왕', en: 'Grand King Kim Mun-mu' }
                ]
            }
        };

        const FACTION_META = {
            goguryeo: { name: '고구려', nameEn: 'Goguryeo', fill: '#b91c1c' },
            baekje:   { name: '백제',   nameEn: 'Baekje',   fill: '#d97706' },
            silla:    { name: '신라',   nameEn: 'Silla',    fill: '#047857' },
            china:    { name: '중원제국', nameEn: 'China Empire', fill: '#581c87' },
            nomad:    { name: '북방초원', nameEn: 'Northern Steppe', fill: '#334155' },
            wa:       { name: '왜열도',   nameEn: 'Wa Archipelago',  fill: '#0e7490' }
        };

        /* =========================================================================
           2. 동아시아 실제 지형 기반 40분획 데이터 (KOR / EN 지명 지원)
           ========================================================================= */
        const PROVINCES_40 = [
            // --- [한반도 및 탐라: 12개 분획 (k01 ~ k12)] ---
            { id: 'k01', name: '평양', nameEn: 'Pyeongyang', zone: '한반도 북부', zoneEn: 'Northern Korea', owner: 'goguryeo', capital: 'goguryeo', label: [484, 294], path: 'M 450,275 L 510,275 L 510,308 L 450,308 Z' },
            { id: 'k02', name: '개마고원', nameEn: 'Gaema Plateau', zone: '한반도 북부', zoneEn: 'Northern Korea', owner: 'goguryeo', label: [524, 266], path: 'M 505,245 L 555,245 L 555,282 L 510,282 L 510,275 L 505,275 Z' },
            { id: 'k03', name: '황해대방', nameEn: 'Hwanghae', zone: '한반도 중부', zoneEn: 'Central Korea', owner: 'goguryeo', label: [484, 316], path: 'M 455,308 L 512,308 L 512,324 L 455,324 Z' },
            { id: 'k04', name: '동해안변', nameEn: 'Anbyeon', zone: '한반도 동부', zoneEn: 'Eastern Korea', owner: 'goguryeo', label: [534, 300], path: 'M 510,282 L 560,282 L 560,318 L 512,318 L 512,308 L 510,308 Z' },
            { id: 'k05', name: '한성', nameEn: 'Hanseong', zone: '한반도 중부', zoneEn: 'Central Korea', owner: 'baekje', capital: 'baekje', label: [492, 335], path: 'M 465,324 L 518,324 L 518,345 L 465,345 Z' },
            { id: 'k06', name: '강원춘천', nameEn: 'Chuncheon', zone: '한반도 동부', zoneEn: 'Eastern Korea', owner: 'silla', label: [538, 332], path: 'M 512,318 L 590,318 L 590,345 L 518,345 L 518,324 L 512,324 Z' },
            { id: 'k07', name: '웅진사비', nameEn: 'Ungjin-Sabi', zone: '한반도 서남', zoneEn: 'SW Korea', owner: 'baekje', label: [492, 355], path: 'M 465,345 L 520,345 L 520,365 L 465,365 Z' },
            { id: 'k08', name: '마한완도', nameEn: 'Mahan-Wando', zone: '한반도 서남', zoneEn: 'SW Korea', owner: 'baekje', label: [494, 375], path: 'M 465,365 L 515,365 L 515,390 L 465,390 Z' },
            { id: 'k09', name: '탐라', nameEn: 'Tamna (Jeju)', zone: '남해 도서', zoneEn: 'Southern Island', owner: 'baekje', label: [490, 412], path: 'M 465,398 L 518,398 L 518,428 L 465,428 Z' },
            { id: 'k10', name: '서라벌', nameEn: 'Seorabeol', zone: '한반도 동남', zoneEn: 'SE Korea', owner: 'silla', capital: 'silla', label: [544, 360], path: 'M 520,345 L 570,345 L 570,385 L 515,385 L 515,365 L 520,365 Z' },
            { id: 'k11', name: '가야금관', nameEn: 'Gaya', zone: '한반도 남부', zoneEn: 'Southern Korea', owner: 'silla', label: [524, 376], path: 'M 512,362 L 535,362 L 535,388 L 512,388 Z' },
            { id: 'k12', name: '상주죽령', nameEn: 'Sangju', zone: '한반도 중남', zoneEn: 'South-Central Korea', owner: 'silla', label: [528, 348], path: 'M 515,340 L 540,340 L 540,356 L 515,356 Z' },

            // --- [만주 및 북방 초원: 9개 분획 (m01 ~ m09)] ---
            { id: 'm01', name: '요서회랑', nameEn: 'Liaoxi', zone: '만주 서부', zoneEn: 'West Manchuria', owner: 'goguryeo', label: [365, 198], path: 'M 330,170 L 395,170 L 395,235 L 330,235 Z' },
            { id: 'm02', name: '요동성', nameEn: 'Liaodong', zone: '만주 중심', zoneEn: 'Central Manchuria', owner: 'goguryeo', label: [422, 224], path: 'M 390,180 L 455,180 L 455,275 L 380,275 Z' },
            { id: 'm03', name: '국내성', nameEn: 'Gungnae', zone: '만주 압록', zoneEn: 'Yalu Basin', owner: 'goguryeo', label: [480, 232], path: 'M 455,190 L 510,190 L 510,275 L 455,275 Z' },
            { id: 'm04', name: '백두산맥', nameEn: 'Mt. Baekdu', zone: '만주 동부', zoneEn: 'East Manchuria', owner: 'goguryeo', label: [542, 218], path: 'M 510,185 L 585,185 L 585,245 L 510,245 Z' },
            { id: 'm05', name: '부여평원', nameEn: 'Buyeo Plain', zone: '북만주', zoneEn: 'North Manchuria', owner: 'nomad', label: [465, 142], path: 'M 405,105 L 520,105 L 520,190 L 405,190 Z' },
            { id: 'm06', name: '송화강', nameEn: 'Songhua Basin', zone: '북만주', zoneEn: 'North Manchuria', owner: 'nomad', label: [575, 132], path: 'M 520,85 L 640,85 L 640,185 L 520,185 Z' },
            { id: 'm07', name: '연해주', nameEn: 'Primorye', zone: '동해 북부', zoneEn: 'Maritime Province', owner: 'nomad', label: [672, 112], path: 'M 640,55 L 780,55 L 780,185 L 640,185 Z' },
            { id: 'm08', name: '흥안령', nameEn: 'Khingan Range', zone: '북방 초원', zoneEn: 'Northern Steppe', owner: 'nomad', label: [345, 112], path: 'M 280,50 L 405,50 L 405,170 L 280,170 Z' },
            { id: 'm09', name: '몽골초원', nameEn: 'Mongol Steppe', zone: '북방 초원', zoneEn: 'Mongol Steppe', owner: 'nomad', label: [165, 95], path: 'M 0,0 L 280,0 L 280,170 L 0,170 Z' },

            // --- [중원 대륙: 12개 분획 (c01 ~ c12)] ---
            { id: 'c01', name: '북평유주', nameEn: 'Beiping', zone: '중원 북부', zoneEn: 'North China', owner: 'china', label: [292, 215], path: 'M 245,170 L 330,170 L 330,255 L 245,255 Z' },
            { id: 'c02', name: '하북업성', nameEn: 'Hebei-Ye', zone: '중원 북부', zoneEn: 'North China', owner: 'china', label: [275, 282], path: 'M 235,255 L 330,255 L 330,310 L 235,310 Z' },
            { id: 'c03', name: '산둥제루', nameEn: 'Shandong', zone: '중원 동해안', zoneEn: 'East China Coast', owner: 'china', label: [358, 284], path: 'M 320,255 L 410,255 L 410,312 L 320,312 Z' },
            { id: 'c04', name: '산서태원', nameEn: 'Shanxi-Taiyuan', zone: '중원 북서', zoneEn: 'NW China', owner: 'china', label: [185, 235], path: 'M 125,170 L 245,170 L 245,295 L 125,295 Z' },
            { id: 'c05', name: '삭방오르도스', nameEn: 'Ordos', zone: '중원 서북', zoneEn: 'NW China', owner: 'china', label: [65, 235], path: 'M 0,170 L 125,170 L 125,295 L 0,295 Z' },
            { id: 'c06', name: '낙양중원', nameEn: 'Luoyang', zone: '중원 중심', zoneEn: 'Central Plains', owner: 'china', label: [215, 335], path: 'M 165,295 L 268,295 L 268,375 L 165,375 Z' },
            { id: 'c07', name: '장안관중', nameEn: 'Changan', zone: '중원 황도', zoneEn: 'Imperial Capital', owner: 'china', label: [95, 338], path: 'M 0,295 L 165,295 L 165,380 L 0,380 Z' },
            { id: 'c08', name: '회수서주', nameEn: 'Huai-Xuzhou', zone: '중원 동부', zoneEn: 'East China', owner: 'china', label: [312, 342], path: 'M 268,310 L 375,310 L 375,372 L 268,372 Z' },
            { id: 'c09', name: '강남건강', nameEn: 'Jiangnan', zone: '중원 강남', zoneEn: 'South Yangtze', owner: 'china', label: [322, 405], path: 'M 275,372 L 380,372 L 380,445 L 275,445 Z' },
            { id: 'c10', name: '형주양양', nameEn: 'Jingzhou', zone: '중원 내륙', zoneEn: 'Central Yangtze', owner: 'china', label: [212, 418], path: 'M 150,375 L 275,375 L 275,465 L 150,465 Z' },
            { id: 'c11', name: '파촉성도', nameEn: 'Sichuan', zone: '중원 서남', zoneEn: 'SW China', owner: 'china', label: [75, 435], path: 'M 0,380 L 150,380 L 150,490 L 0,490 Z' },
            { id: 'c12', name: '영남광주', nameEn: 'Lingnan', zone: '중원 남해', zoneEn: 'South China', owner: 'china', label: [205, 505], path: 'M 0,465 L 360,465 L 360,560 L 0,560 Z' },

            // --- [왜 열도: 7개 분획 (w01 ~ w07)] ---
            { id: 'w01', name: '대마도', nameEn: 'Tsushima', zone: '대한해협', zoneEn: 'Korea Strait', owner: 'wa', label: [562, 392], path: 'M 550,376 L 576,376 L 576,406 L 550,406 Z' },
            { id: 'w02', name: '북규슈', nameEn: 'North Kyushu', zone: '왜 열도 서부', zoneEn: 'West Archipelago', owner: 'wa', label: [595, 418], path: 'M 568,400 L 620,400 L 620,435 L 568,435 Z' },
            { id: 'w03', name: '남규슈', nameEn: 'South Kyushu', zone: '왜 열도 서남', zoneEn: 'SW Archipelago', owner: 'wa', label: [592, 452], path: 'M 568,435 L 620,435 L 620,478 L 568,478 Z' },
            { id: 'w04', name: '시코쿠', nameEn: 'Shikoku', zone: '왜 열도 내해', zoneEn: 'Inland Sea', owner: 'wa', label: [642, 418], path: 'M 620,402 L 666,402 L 666,438 L 620,438 Z' },
            { id: 'w05', name: '주고쿠이즈모', nameEn: 'Chugoku-Izumo', zone: '혼슈 서부', zoneEn: 'West Honshu', owner: 'wa', label: [628, 388], path: 'M 594,368 L 656,368 L 656,402 L 594,402 Z' },
            { id: 'w06', name: '아스카기나이', nameEn: 'Asuka-Kinai', zone: '왜 왕궁 중심', zoneEn: 'Yamato Court', owner: 'wa', label: [684, 384], path: 'M 656,342 L 712,342 L 712,415 L 656,415 Z' },
            { id: 'w07', name: '간토동부', nameEn: 'Kanto Plain', zone: '혼슈 동부', zoneEn: 'East Honshu', owner: 'wa', label: [738, 338], path: 'M 712,260 L 770,260 L 770,408 L 712,408 Z' }
        ];

        const EXPANSION_PATHS = {
            korea:  ['k05', 'k03', 'k07', 'k06', 'k12', 'k01', 'k10', 'k11', 'k08', 'k04', 'k09', 'k02', 'm03', 'm02'],
            china:  ['m01', 'c03', 'c01', 'c02', 'c08', 'c06', 'c09', 'c04', 'c07', 'c10', 'c05', 'c11', 'c12'],
            wa:     ['w01', 'w02', 'w03', 'w05', 'w04', 'w06', 'w07'],
            steppe: ['m05', 'm04', 'm06', 'm07', 'm08', 'm09', 'c04', 'c05'],
            any:    ['k05', 'k07', 'c03', 'w01', 'm05', 'k01', 'k10', 'c01', 'w02', 'c06', 'w06', 'c07', 'm09']
        };

        /* =========================================================================
           3. 차등 확률 + 성공/실패 분기 + 50~100턴 대장정 이벤트 생성 (KOR / EN 지원)
           ========================================================================= */
        const CORE_EVENTS = [
            {
                tag: '삼한 쟁패 · 한강전선', tagEn: 'Korean Peninsula · Han River Front',
                title: '한반도의 중심 한강 유역과 서해 포구 총공세',
                titleEn: 'Grand Offensive for the Han River Basin and West Sea Ports',
                desc: '한강 유역과 서해안 나루터는 삼한 통일의 심장이자 중원 대륙으로 나아가는 전략 요충지입니다. 어떤 전략으로 중부 전선을 돌파하시겠습니까?',
                descEn: 'The Han River basin and western ports are the heart of Korean unification and the gateway to China. How will you break through the central front?',
                choices: [
                    {
                        text: '주력 철기군을 총동원해 한반도 중부 핵심 두 구역을 정면 돌파한다',
                        textEn: 'Mobilize our main iron cavalry to storm two central regions head-on',
                        baseProb: 58, statKey: 'military',
                        winEff: { economy: -4, military: -4, authority: +12, diplomacy: -2 },
                        failEff: { economy: -8, military: -14, authority: -8, diplomacy: -4 },
                        winTerr: { dir: 'korea', gain: 2 },
                        failTerr: null,
                        winMsg: '철기군 선봉대가 적의 방어선을 정면 돌파하여 한반도 중부 요충지 두 곳을 단숨에 점령했습니다!',
                        winMsgEn: 'Our iron cavalry vanguard shattered the enemy lines and captured two strategic central provinces!',
                        failMsg: '적의 험준한 산성 방어에 막혀 공성전이 길어지면서 병력만 잃고 회군했습니다.',
                        failMsgEn: 'Stalled by rugged mountain fortresses, our siege dragged on and forced a costly retreat.'
                    },
                    {
                        text: '적국 성주를 황금과 관직으로 회유하여 거점 한 곳을 무혈 인수한다',
                        textEn: 'Bribe the enemy fortress lord with gold and rank to take one province bloodlessly',
                        baseProb: 74, statKey: 'diplomacy',
                        winEff: { economy: -8, military: +4, authority: +8, diplomacy: +10 },
                        failEff: { economy: -12, military: -2, authority: -6, diplomacy: -8 },
                        winTerr: { dir: 'korea', gain: 1 },
                        failTerr: null,
                        winMsg: '은밀한 밀사 공작이 적중하여 적 성주가 스스로 성문을 열고 투항했습니다!',
                        winMsgEn: 'Secret envoys succeeded! The enemy commander opened the fortress gates and surrendered peacefully.',
                        failMsg: '적 조정에 밀정이 발각되어 보낸 황금만 몰수당하고 외교적 망신을 당했습니다.',
                        failMsgEn: 'Our secret envoy was caught by the enemy court, losing the gold and suffering diplomatic disgrace.'
                    },
                    {
                        text: '공세를 멈추고 산성을 증축하며 둔전을 일궈 국고와 병력을 크게 채운다',
                        textEn: 'Halt offensives, fortify mountain castles, and expand farmlands to boost power',
                        baseProb: 88, statKey: 'economy',
                        winEff: { economy: +14, military: +12, authority: +8, diplomacy: +4 },
                        failEff: { economy: +4, military: +2, authority: -4, diplomacy: 0 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: '대풍년과 산성 보수가 순조롭게 이루어져 군량미와 병력이 가득 찼습니다.',
                        winMsgEn: 'Bountiful harvests and fortress upgrades filled our granaries and strengthened our army.',
                        failMsg: '장마로 산성 공사가 지연되어 기대만큼 군량미를 비축하지 못했습니다.',
                        failMsgEn: 'Heavy monsoon rains delayed fortress construction, yielding smaller grain reserves than expected.'
                    },
                    {
                        text: '[극단적 승부수] 전 국력을 쏟아부어 적의 본토 세 구역을 동시에 기습 정벌한다!',
                        textEn: '[All-Out Gamble] Stake the entire nation to blitz three enemy heartland regions at once!',
                        baseProb: 38, statKey: 'military',
                        winEff: { economy: -8, military: -8, authority: +20, diplomacy: -6 },
                        failEff: { economy: -14, military: -16, authority: -12, diplomacy: -8 },
                        winTerr: { dir: 'korea', gain: 3 },
                        failTerr: { dir: 'korea', gain: 0, lose: 1, enemy: 'china' },
                        winMsg: '역사에 남을 대담한 기습 작전 대성공! 단숨에 세 구역을 정벌하며 삼한의 패권을 장악했습니다!',
                        winMsgEn: 'A legendary blitzkrieg victory! Three provinces fell at once, establishing supreme hegemony!',
                        failMsg: '무리한 삼면 기습이 적의 매복에 걸려 대패하고 오히려 국경 거점 한 곳을 빼앗겼습니다!',
                        failMsgEn: 'Our overextended three-pronged attack fell into an ambush, losing a border province in defeat!'
                    }
                ]
            },
            {
                tag: '대륙 원정 · 황해 도하', tagEn: 'Continental Campaign · Yellow Sea Crossing',
                title: '요서 회랑 진격과 산둥반도 대규모 상륙 작전',
                titleEn: 'Advance into Liaoxi Corridor & Amphibious Landing on Shandong',
                desc: '중원 왕조가 국경을 압박하고 있습니다. 수세에 몰리기 전에 육군으로 요서와 북평을 치고, 대함대로 서해를 건너 산둥반도를 정벌할 기회입니다!',
                descEn: 'Chinese dynasties are pressuring our borders. Strike Liaoxi and Beiping by land and cross the Yellow Sea to conquer Shandong Peninsula!',
                choices: [
                    {
                        text: '육군과 수군을 동시에 출격시켜 산둥반도와 북평 일대 두 구역을 정벌한다!',
                        textEn: 'Launch joint land and naval forces to conquer both Shandong and Beiping!',
                        baseProb: 54, statKey: 'military',
                        winEff: { economy: -6, military: -6, authority: +15, diplomacy: -4 },
                        failEff: { economy: -12, military: -14, authority: -8, diplomacy: -6 },
                        winTerr: { dir: 'china', gain: 2, pick: ['c03', 'c01', 'm01'] },
                        failTerr: null,
                        winMsg: '우리 수군이 산둥반도를 장악하고 육군이 만리장성 너머 북평을 점령했습니다!',
                        winMsgEn: 'Our navy seized Shandong Peninsula while our army marched past the Great Wall into Beiping!',
                        failMsg: '서해 풍랑과 중원군의 완강한 저항으로 상륙군이 교두보를 확보하지 못하고 철수했습니다.',
                        failMsgEn: 'Fierce storms and stubborn imperial defenses forced our landing forces to withdraw.'
                    },
                    {
                        text: '산둥 해안에 무장 상단을 보내 거점 한 곳을 복속시키고 황해 무역을 독점한다',
                        textEn: 'Dispatch armed merchant fleets to secure a Shandong port and monopolize sea trade',
                        baseProb: 72, statKey: 'economy',
                        winEff: { economy: +14, military: -2, authority: +8, diplomacy: +10 },
                        failEff: { economy: -10, military: -6, authority: -4, diplomacy: -6 },
                        winTerr: { dir: 'china', gain: 1, pick: ['c03', 'c08'] },
                        failTerr: null,
                        winMsg: '산둥 해안 무역 거점을 완전히 장악하고 막대한 해상 관세를 거두어들입니다!',
                        winMsgEn: 'We secured a vital Shandong trade port, bringing massive maritime customs into our treasury!',
                        failMsg: '중원 해안 수비대의 기습으로 상선단이 피해를 입고 무역 기지 건설에 실패했습니다.',
                        failMsgEn: 'Chinese coastal guards ambushed our merchant fleet, halting the trade outpost plan.'
                    },
                    {
                        text: '중원 조정에 사절단을 보내 평화 조약을 맺고 비단과 제철 기술을 들여온다',
                        textEn: 'Send envoys to conclude a peace treaty and import silk and advanced ironworking',
                        baseProb: 86, statKey: 'diplomacy',
                        winEff: { economy: +15, military: +8, authority: +6, diplomacy: +14 },
                        failEff: { economy: -4, military: +2, authority: -6, diplomacy: -4 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: '능수능란한 외교 담판으로 전쟁 없이 선진 철기 기술과 막대한 비단을 확보했습니다.',
                        winMsgEn: 'Skillful diplomacy secured advanced metallurgy and silk without shedding a drop of blood.',
                        failMsg: '중원 조정의 오만한 요구로 회담이 결렬되어 사절단이 빈손으로 귀환했습니다.',
                        failMsgEn: 'Arrogant demands from the imperial court broke off talks, and our envoys returned empty-handed.'
                    },
                    {
                        text: '[대륙 직격 승부수] 산둥과 하북을 넘어 중원 중심부 낙양까지 세 구역을 휘젓는다!',
                        textEn: '[Continental Blitz] Storm across Shandong and Hebei straight into Luoyang to seize three regions!',
                        baseProb: 35, statKey: 'military',
                        winEff: { economy: -10, military: -10, authority: +22, diplomacy: -8 },
                        failEff: { economy: -16, military: -16, authority: -12, diplomacy: -10 },
                        winTerr: { dir: 'china', gain: 3, pick: ['c03', 'c02', 'c06'] },
                        failTerr: { dir: 'china', gain: 0, lose: 1, enemy: 'china' },
                        winMsg: '황하를 건넌 원정군이 낙양성까지 파죽지세로 함락시키며 중원 대륙을 경악시켰습니다!',
                        winMsgEn: 'Crossing the Yellow River, our army captured Luoyang itself, shocking all of China!',
                        failMsg: '깊숙이 진격한 원정군의 보급선이 끊기며 대패하여 변경 영토 한 곳을 상실했습니다.',
                        failMsgEn: 'Our deep expedition had its supply lines severed, suffering defeat and losing a border region.'
                    }
                ]
            },
            {
                tag: '대체역사 · 중원 천하통일', tagEn: 'Alternate History · Conquest of China',
                title: '중원 황제의 도읍 장안성 포위와 강남 대평정',
                titleEn: 'Siege of Imperial Capital Changan & Pacification of Jiangnan',
                desc: '우리 원정군이 황하와 회수를 넘어 중원의 심장부로 진격하고 있습니다! 서쪽의 황제 수도 장안과 부유한 강남까지 정벌하여 대륙의 주인이 되시겠습니까?',
                descEn: 'Our armies have crossed the Yellow and Huai Rivers! Will you march on Changan and wealthy Jiangnan to become ruler of the continent?',
                choices: [
                    {
                        text: '장안성과 강남 건강까지 진격하여 중원 대륙 핵심 두 구역을 병합한다!',
                        textEn: 'March on Changan and Jiangnan to annex two core continental regions!',
                        baseProb: 52, statKey: 'military',
                        winEff: { economy: +8, military: -8, authority: +16, diplomacy: +4 },
                        failEff: { economy: -12, military: -14, authority: -10, diplomacy: -6 },
                        winTerr: { dir: 'china', gain: 2, pick: ['c07', 'c09', 'c06'] },
                        failTerr: null,
                        winMsg: '장안성과 강남에 우리 왕조의 군기가 나부끼며 중원 대륙의 중심부가 복속되었습니다!',
                        winMsgEn: 'Our royal banners now fly over Changan and Jiangnan as the continental heartland submits!',
                        failMsg: '장안성의 험준한 관중 요새를 뚫지 못하고 공성 병기만 소모한 채 물러났습니다.',
                        failMsgEn: 'Unable to breach the formidable passes of Changan, our siege engines were exhausted.'
                    },
                    {
                        text: '부유한 양쯔강 하류 강남 곡창지대 한 곳을 집중 공략해 제국의 금고로 삼는다',
                        textEn: 'Focus on capturing the rich Yangtze delta of Jiangnan as our imperial treasury',
                        baseProb: 70, statKey: 'economy',
                        winEff: { economy: +18, military: -4, authority: +10, diplomacy: +6 },
                        failEff: { economy: -8, military: -8, authority: -6, diplomacy: -4 },
                        winTerr: { dir: 'china', gain: 1, pick: ['c09', 'c08', 'c12'] },
                        failTerr: null,
                        winMsg: '강남의 비옥한 곡창지대를 점령하여 매년 수십만 석의 세곡이 도성으로 들어옵니다!',
                        winMsgEn: 'Capturing the fertile Jiangnan granaries brings ratusan of thousands of grain sacks annually!',
                        failMsg: '강남 수군의 화공에 막혀 양쯔강 도하 작전이 좌절되었습니다.',
                        failMsgEn: 'Fire attacks by the southern river navy foiled our Yangtze crossing operation.'
                    },
                    {
                        text: '중원 황제에게 항복 문서와 막대한 세폐만 받고 군대를 물려 전력을 보존한다',
                        textEn: 'Accept massive tribute and a surrender treaty from the Emperor to preserve our forces',
                        baseProb: 85, statKey: 'diplomacy',
                        winEff: { economy: +20, military: +10, authority: +10, diplomacy: +14 },
                        failEff: { economy: +4, military: 0, authority: -6, diplomacy: -4 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: '중원 황제가 해마다 황금과 비단 수만 필을 바치기로 맹세하며 국력이 크게 신장됩니다.',
                        winMsgEn: 'The Chinese Emperor swore annual tribute of gold and silk, greatly boosting our national power.',
                        failMsg: '중원 측이 화친 회담을 시간 끌기용으로 악용하여 소규모 배상금만 받는 데 그쳤습니다.',
                        failMsgEn: 'The imperial court used peace talks to stall for time, yielding only minor reparations.'
                    },
                    {
                        text: '[대륙 전역 석권] 파촉과 영남 해안까지 세 구역을 단숨에 휩쓸어 천하를 통일한다!',
                        textEn: '[Total Continental Conquest] Sweep through Sichuan and Lingnan to conquer three regions!',
                        baseProb: 34, statKey: 'military',
                        winEff: { economy: +12, military: -12, authority: +24, diplomacy: +6 },
                        failEff: { economy: -16, military: -18, authority: -14, diplomacy: -10 },
                        winTerr: { dir: 'china', gain: 3, pick: ['c07', 'c10', 'c11', 'c12'] },
                        failTerr: { dir: 'china', gain: 0, lose: 1, enemy: 'china' },
                        winMsg: '전무후무한 대정벌 성공! 중원 대륙 세 구역을 동시에 평정하며 동방의 대제국으로 우뚝 섰습니다!',
                        winMsgEn: 'Unprecedented triumph! Three continental regions fell at once, forging a supreme Eastern Empire!',
                        failMsg: '험준한 촉도와 남방 풍토병으로 원정군이 큰 피해를 입고 점령지 한 곳에서 후퇴했습니다.',
                        failMsgEn: 'Treacherous mountain roads and tropical disease forced a costly retreat from one province.'
                    }
                ]
            },
            {
                tag: '해상 원정 · 왜 열도 정벌', tagEn: 'Naval Expedition · Archipelago Campaign',
                title: '대마도 제압과 규슈·아스카 열도 상륙 대작전',
                titleEn: 'Subjugation of Tsushima & Grand Landing on Kyushu and Asuka',
                desc: '대한해협 건너 왜 열도의 세력이 해안을 위협합니다. 대함대를 출동시켜 대마도와 규슈를 치고, 세토 내해를 돌파해 아스카 왕궁까지 정벌하시겠습니까?',
                descEn: 'Forces across the Korea Strait threaten our coasts. Will you send a grand fleet to conquer Tsushima, Kyushu, and the Asuka court?',
                choices: [
                    {
                        text: '전함 오백 척으로 대마도와 규슈 열도 서부 두 구역을 정벌한다!',
                        textEn: 'Deploy 500 warships to conquer Tsushima and western Kyushu!',
                        baseProb: 56, statKey: 'military',
                        winEff: { economy: +6, military: -6, authority: +14, diplomacy: +2 },
                        failEff: { economy: -10, military: -12, authority: -8, diplomacy: -4 },
                        winTerr: { dir: 'wa', gain: 2, pick: ['w01', 'w02', 'w03'] },
                        failTerr: null,
                        winMsg: '대한해협과 규슈 열도가 우리 수군의 직할 영토로 편입되었습니다!',
                        winMsgEn: 'The Korea Strait and Kyushu have been incorporated into our direct naval territory!',
                        failMsg: '대한해협의 거센 해류와 폭풍우로 함선 일부가 파손되어 회항했습니다.',
                        failMsgEn: 'Fierce currents and typhoons in the strait damaged our fleet and forced a return.'
                    },
                    {
                        text: '대마도와 북규슈 중 한 곳에 교역 담로를 설치하고 왜 호족의 조공을 받는다',
                        textEn: 'Establish a trade province in Tsushima or North Kyushu and receive tribute from local clans',
                        baseProb: 75, statKey: 'diplomacy',
                        winEff: { economy: +12, military: +4, authority: +8, diplomacy: +14 },
                        failEff: { economy: -6, military: -4, authority: -4, diplomacy: -6 },
                        winTerr: { dir: 'wa', gain: 1, pick: ['w01', 'w02'] },
                        failTerr: null,
                        winMsg: '왜 열도 서부 호족들이 우리 왕실의 작위를 받고 은과 진주를 조공으로 바칩니다!',
                        winMsgEn: 'Western archipelago clans accepted our royal titles and offered silver and pearls in tribute!',
                        failMsg: '열도 호족들의 내분으로 사절단이 협상을 마무리하지 못하고 귀환했습니다.',
                        failMsgEn: 'Internal strife among archipelago clans prevented our envoys from sealing the pact.'
                    },
                    {
                        text: '원정 대신 남해안과 탐라에 대형 장갑선을 배치하여 해상 방어력을 높인다',
                        textEn: 'Station armored warships along the southern coast and Tamna to bolster naval defense',
                        baseProb: 90, statKey: 'authority',
                        winEff: { economy: +8, military: +14, authority: +10, diplomacy: +6 },
                        failEff: { economy: -2, military: +4, authority: 0, diplomacy: 0 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: '철통같은 남해안 수군 방어망이 구축되어 해적들이 그림자도 비치지 못합니다.',
                        winMsgEn: 'An ironclad naval defense line now protects our southern seas from any raiders.',
                        failMsg: '목재 수급 지연으로 전함 건조가 일부 늦어졌으나 해안 방어는 유지되었습니다.',
                        failMsgEn: 'Timber shortages slightly delayed warship construction, though coastal defenses held firm.'
                    },
                    {
                        text: '[열도 전역 정벌] 세토 내해를 돌파해 아스카 기나이와 간토까지 세 구역을 점령한다!',
                        textEn: '[Archipelago Conquest] Push through the Inland Sea to conquer Asuka and Kanto!',
                        baseProb: 36, statKey: 'military',
                        winEff: { economy: +10, military: -10, authority: +22, diplomacy: -4 },
                        failEff: { economy: -14, military: -16, authority: -10, diplomacy: -8 },
                        winTerr: { dir: 'wa', gain: 3, pick: ['w05', 'w06', 'w07'] },
                        failTerr: { dir: 'wa', gain: 0, lose: 1, enemy: 'wa' },
                        winMsg: '아스카 왕궁이 항복하고 간토 평야까지 왜 열도 전역에 우리 왕조의 총독부가 세워졌습니다!',
                        winMsgEn: 'The Asuka court surrendered, and our royal governors now rule all the way to the Kanto plain!',
                        failMsg: '세토 내해의 좁은 해협에서 기습을 받아 원정 함대가 후퇴하고 해안 거점 한 곳을 잃었습니다.',
                        failMsgEn: 'Ambushed in the narrow Inland Sea straits, our fleet retreated and lost a coastal outpost.'
                    }
                ]
            },
            {
                tag: '북방 개척 · 초원 대가한', tagEn: 'Northern Frontier · Steppe Khaganate',
                title: '흥안령과 송화강 돌파, 북방 대초원 유목 제국 복속',
                titleEn: 'Crossing the Songhua & Khingan Range to Subdue the Steppe',
                desc: '만주 북방의 부여·말갈과 초원의 거란·돌궐 기마대가 움직이고 있습니다. 이들을 정벌하여 복속시키면 초원의 무적 기병대를 우리 군대로 삼을 수 있습니다!',
                descEn: 'Nomadic horsemen of Manchuria and the Mongol steppe are stirring. Subduing them will add invincible steppe cavalry to our empire!',
                choices: [
                    {
                        text: '중장기병을 몰아 송화강과 연해주, 부여 평원 중 두 구역을 무력으로 평정한다!',
                        textEn: 'Lead heavy cavalry to conquer two regions across Songhua, Primorye, and Buyeo!',
                        baseProb: 58, statKey: 'military',
                        winEff: { economy: +4, military: +10, authority: +14, diplomacy: 0 },
                        failEff: { economy: -8, military: -12, authority: -8, diplomacy: -4 },
                        winTerr: { dir: 'steppe', gain: 2, pick: ['m05', 'm06', 'm07'] },
                        failTerr: null,
                        winMsg: '광활한 북만주 벌판과 연해주가 평정되고 숙신·말갈 기병대가 우리 군에 편입되었습니다!',
                        winMsgEn: 'Northern Manchuria and Primorye are pacified, integrating elite steppe horsemen into our army!',
                        failMsg: '혹독한 북방 추위와 유목민의 치고 빠지는 전술에 말려 소득 없이 철군했습니다.',
                        failMsgEn: 'Bitter northern frost and nomadic hit-and-run tactics forced our expedition to withdraw.'
                    },
                    {
                        text: '초원 부족장들에게 철제 무기와 곡식을 하사하여 한 구역을 평화롭게 복속시킨다',
                        textEn: 'Grant iron weapons and grain to steppe chieftains to peacefully annex one region',
                        baseProb: 76, statKey: 'diplomacy',
                        winEff: { economy: -4, military: +12, authority: +8, diplomacy: +14 },
                        failEff: { economy: -10, military: -2, authority: -4, diplomacy: -6 },
                        winTerr: { dir: 'steppe', gain: 1, pick: ['m05', 'm08'] },
                        failTerr: null,
                        winMsg: '초원 부족장들이 전하의 은덕에 감복하여 스스로 영토를 바치고 기마병을 보냈습니다!',
                        winMsgEn: 'Steppe chieftains pledged allegiance, offering their lands and auxiliary cavalry!',
                        failMsg: '물자만 챙긴 유목 부족이 약속을 어기고 초원 깊숙이 달아났습니다.',
                        failMsgEn: 'Nomadic tribes took our supplies and fled deep into the steppe, breaking their oath.'
                    },
                    {
                        text: '백두산과 요동 방어선에 천리장성을 쌓고 북방 철광산을 집중 개발한다',
                        textEn: 'Build a Great Wall along Liaodong and Mt. Baekdu and develop northern iron mines',
                        baseProb: 88, statKey: 'economy',
                        winEff: { economy: +14, military: +12, authority: +8, diplomacy: +4 },
                        failEff: { economy: +2, military: +4, authority: -2, diplomacy: 0 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: '요동과 백두산 방어선이 철옹성이 되고 강철 갑옷과 창검이 대량 생산됩니다!',
                        winMsgEn: 'Our northern frontier became impregnable while northern mines mass-produce steel armor!',
                        failMsg: '북방 광산 갱도 일부가 무너져 철광석 생산량이 예상보다 줄었습니다.',
                        failMsgEn: 'A tunnel collapse in the northern mines reduced iron ore output below expectations.'
                    },
                    {
                        text: '[초원 끝까지 정벌] 흥안령 너머 몽골 초원까지 북아시아 세 구역을 단숨에 제패한다!',
                        textEn: '[Steppe Domination] Ride beyond Khingan into the Mongol Steppe to conquer three regions!',
                        baseProb: 37, statKey: 'military',
                        winEff: { economy: +6, military: +16, authority: +20, diplomacy: -4 },
                        failEff: { economy: -12, military: -16, authority: -12, diplomacy: -8 },
                        winTerr: { dir: 'steppe', gain: 3, pick: ['m06', 'm08', 'm09'] },
                        failTerr: { dir: 'steppe', gain: 0, lose: 1, enemy: 'nomad' },
                        winMsg: '몽골 대초원의 부족들이 모두 무릎을 꿇고 전하를 대초원의 지배자로 추대했습니다!',
                        winMsgEn: 'All tribes of the Mongol Steppe knelt and proclaimed Your Majesty the Supreme Khagan!',
                        failMsg: '끝없는 초원에서 길을 잃은 원정군이 유목 연합군의 포위망에 걸려 변경 한 곳을 내주었습니다.',
                        failMsgEn: 'Lost in the vast steppe, our expedition was encircled by nomadic coalitions, losing a border zone.'
                    }
                ]
            }
        ];

        // 파생 발생 이벤트(연쇄 사건) 풀 (KOR / EN 지원)
        const CHAIN_EVENTS = {
            win: [
                {
                    tag: '승전 파생 호재', tagEn: 'Victory Bonus',
                    title: '점령지 기술자와 장인들의 집단 귀순', titleEn: 'Artisans and Blacksmiths Join Our Realm',
                    desc: '새로 확보한 지역과 교역로를 통해 숙련된 대장장이와 조선공들이 도성으로 들어와 국력이 추가로 상승합니다.',
                    descEn: 'Skilled blacksmiths and shipwrights from newly secured territories flocked to our capital, boosting national power.',
                    eff: { economy: +4, military: +4 }
                },
                {
                    tag: '민심 고양', tagEn: 'Soaring Morale',
                    title: '도성 저잣거리의 승전 잔치와 자원입대 행렬', titleEn: 'Capital Victory Festival & Volunteer Recruits',
                    desc: '조정의 통쾌한 결단 소식에 백성들의 사기가 충천하여 청년들이 앞다투어 군영에 지원합니다.',
                    descEn: 'Inspired by the royal triumph, citizens celebrate in the streets and youths volunteer for the army.',
                    eff: { authority: +5, military: +3 }
                },
                {
                    tag: '외교 사절 쇄도', tagEn: 'Foreign Envoys Arrive',
                    title: '주변 소국과 부족장들의 조공 사절단 도착', titleEn: 'Tribute Missions from Neighboring States',
                    desc: '우리 왕조의 위세가 높아지자 이웃 세력들이 다투어 말과 비단을 바치며 화친을 청해 옵니다.',
                    descEn: 'Awed by our rising prestige, neighboring states sent horses and silk to seek peace.',
                    eff: { economy: +5, diplomacy: +5 }
                }
            ],
            fail: [
                {
                    tag: '패전 후유증', tagEn: 'Setback Aftermath',
                    title: '변경 유민 발생과 보급로 혼선', titleEn: 'Border Refugees & Supply Disruption',
                    desc: '작전 차질로 인해 국경 마을 백성들이 동요하고 추가 복구 비용이 지출되었습니다.',
                    descEn: 'Operational setbacks unsettled border towns and required extra recovery funds.',
                    eff: { economy: -3, authority: -3 }
                },
                {
                    tag: '조정 정쟁 발생', tagEn: 'Court Factional Dispute',
                    title: '정무 실패 책임을 둘러싼 귀족 회의의 분열', titleEn: 'Noble Council Divided Over Responsibility',
                    desc: '실패의 책임을 두고 대신들 사이에 언쟁이 벌어져 국정 운영력이 일시적으로 흔들립니다.',
                    descEn: 'Nobles argued over blame for the setback, temporarily shaking royal authority.',
                    eff: { authority: -4, diplomacy: -2 }
                }
            ]
        };

        /* =========================================================================
           4. 게임 상태, 언어 전환(KOR/EN), 60초 타이머 & 1~8배속 컨트롤러
           ========================================================================= */
        let state = {
            kingdomId: 'goguryeo',
            rulerObj: { ko: '고담덕 태왕', en: 'Grand King Go Dam-deok' },
            year: 391,
            turn: 1,
            maxTurn: 80,
            stats: { economy: 60, military: 70, authority: 60, diplomacy: 55 },
            provinces: {},
            deck: [],
            currentEvent: null,
            lastPopupData: null,
            isOver: false,
            isPopupOpen: false,
            timeLeft: 60.0,
            maxTime: 60.0,
            speedMultiplier: 1,
            lastFrameTime: performance.now()
        };

        let bannerTick = 0;

        // KOR / EN 언어 전환 함수
        function setLanguage(lang) {
            currentLang = (lang === 'en') ? 'en' : 'ko';

            const btnKo = document.getElementById('btn-lang-ko');
            const btnEn = document.getElementById('btn-lang-en');
            if (currentLang === 'ko') {
                btnKo.className = 'px-2 py-1 rounded-md bg-amber-500 text-black transition';
                btnEn.className = 'px-2 py-1 rounded-md text-gray-400 hover:text-white transition';
            } else {
                btnEn.className = 'px-2 py-1 rounded-md bg-amber-500 text-black transition';
                btnKo.className = 'px-2 py-1 rounded-md text-gray-400 hover:text-white transition';
            }

            // 1) 인트로 타이틀 화면 텍스트 번역
            document.title = currentLang === 'ko' ? '한반도 삼국통일 시뮬레이터' : 'Three Kingdoms Unification Simulator';
            document.getElementById('intro-main-title').textContent = currentLang === 'ko'
                ? '한반도 삼국통일 시뮬레이터'
                : 'Three Kingdoms Unification Simulator';
            document.getElementById('intro-sub-desc').textContent = currentLang === 'ko'
                ? '고구려 · 백제 · 신라 삼국의 무작위 군주가 되어 한반도와 만주, 중원 대륙, 왜 열도 40개 구역을 제패하는 대장정 역사 시뮬레이션'
                : 'Become a random monarch of Goguryeo, Baekje, or Silla and conquer 40 regions across Korea, Manchuria, China, and Japan in a grand historical simulation!';
            document.getElementById('intro-k1-name').textContent = currentLang === 'ko' ? '고구려' : 'Goguryeo';
            document.getElementById('intro-k1-sub').textContent = currentLang === 'ko' ? '북방 철기병 · 대륙 정벌' : 'Iron Cavalry · Continental Power';
            document.getElementById('intro-k2-name').textContent = currentLang === 'ko' ? '백제' : 'Baekje';
            document.getElementById('intro-k2-sub').textContent = currentLang === 'ko' ? '해상 대선단 · 열도 담로' : 'Grand Fleet · Maritime Trade';
            document.getElementById('intro-k3-name').textContent = currentLang === 'ko' ? '신라' : 'Silla';
            document.getElementById('intro-k3-sub').textContent = currentLang === 'ko' ? '화랑도 결속 · 삼한 일통' : 'Hwarang Unity · Unification';
            document.getElementById('lbl-intro-start').textContent = currentLang === 'ko' ? '시작하기' : 'START GAME';
            document.getElementById('lbl-visit-daily').textContent = currentLang === 'ko' ? '일일 방문자' : 'TODAY';
            document.getElementById('lbl-visit-total').textContent = currentLang === 'ko' ? '누적 방문자' : 'TOTAL VISITORS';
            updateIntroSceneBadge();

            // 2) 상단 및 고정 라벨 번역
            document.getElementById('lbl-hdr-terr').textContent = currentLang === 'ko' ? '지배 영토:' : 'Territories:';
            document.getElementById('lbl-hdr-terr-unit').textContent = currentLang === 'ko' ? '/ 40분획' : '/ 40 Regions';
            document.getElementById('lbl-turn-goal').textContent = currentLang === 'ko' ? `대장정 ${state.maxTurn}턴` : `Campaign ${state.maxTurn}T`;
            document.getElementById('lbl-new-game').textContent = currentLang === 'ko' ? '새 운명 시작' : 'New Destiny';

            document.getElementById('lbl-stat-eco').textContent = currentLang === 'ko' ? '국고' : 'Treasury';
            document.getElementById('lbl-stat-mil').textContent = currentLang === 'ko' ? '군사' : 'Military';
            document.getElementById('lbl-stat-aut').textContent = currentLang === 'ko' ? '민심' : 'Support';
            document.getElementById('lbl-stat-dip').textContent = currentLang === 'ko' ? '외교' : 'Diplomacy';

            document.getElementById('lbl-timer-title').textContent = currentLang === 'ko' ? '결단 제한 시간' : 'Decision Timer';
            document.getElementById('speed-label').textContent = currentLang === 'ko' ? `${state.speedMultiplier}배속` : `${state.speedMultiplier}x Speed`;
            document.getElementById('speed-sub-hint').textContent = currentLang === 'ko' ? '(클릭 시 +1배속)' : '(Click +1x)';

            document.getElementById('lgd-goguryeo').textContent = currentLang === 'ko' ? '고구려' : 'Goguryeo';
            document.getElementById('lgd-baekje').textContent = currentLang === 'ko' ? '백제' : 'Baekje';
            document.getElementById('lgd-silla').textContent = currentLang === 'ko' ? '신라' : 'Silla';
            document.getElementById('lgd-china').textContent = currentLang === 'ko' ? '중원' : 'China';
            document.getElementById('lgd-nomad').textContent = currentLang === 'ko' ? '초원' : 'Steppe';
            document.getElementById('lgd-wa').textContent = currentLang === 'ko' ? '왜열도' : 'Wa';

            document.getElementById('map-hover-info').textContent = currentLang === 'ko'
                ? '실선: 정밀 해안선·강줄기 | 점선: 40분획 경계 (선택지에 마우스를 올리면 목표 구역 표시)'
                : 'Solid: Coastlines & Rivers | Dashed: 40 Regions (Hover over choices to preview targets)';

            document.getElementById('sea-bohai').textContent = currentLang === 'ko' ? '발해만' : 'Bohai Bay';
            document.getElementById('sea-west').textContent = currentLang === 'ko' ? '서해 (황해)' : 'West Sea (Yellow Sea)';
            document.getElementById('sea-east').textContent = currentLang === 'ko' ? '동해' : 'East Sea';
            document.getElementById('sea-strait').textContent = currentLang === 'ko' ? '대한해협' : 'Korea Strait';
            document.getElementById('sea-ecs').textContent = currentLang === 'ko' ? '동중국해' : 'East China Sea';
            document.getElementById('sea-pacific').textContent = currentLang === 'ko' ? '태평양' : 'Pacific Ocean';

            document.getElementById('lbl-popup-sec1').textContent = currentLang === 'ko' ? '선택 결단 및 실행 결과' : 'Decision & Execution Outcome';
            document.getElementById('lbl-popup-sec2').textContent = currentLang === 'ko' ? '국력 및 40분획 영토 변동' : 'National Power & Territory Changes';
            document.getElementById('lbl-popup-btn').textContent = currentLang === 'ko' ? '결과 확인 및 다음 해 정무로 진행' : 'Confirm & Proceed to Next Year';
            document.getElementById('btn-ending-restart').textContent = currentLang === 'ko'
                ? '새로운 무작위 왕조와 군주로 대장정 다시 시작'
                : 'Start a New Campaign with a Random Kingdom & Monarch';

            buildMapSVG();
            refreshMapSVG();
            refreshUI();
            if (state.currentEvent && !state.isOver) {
                renderCurrentEventView();
            }
            if (state.isPopupOpen && state.lastPopupData) {
                renderPopupContent(state.lastPopupData);
            }
        }

        // 1배속 -> 2배속 -> ... -> 8배속 -> 1배속 순환 함수
        function cycleGameSpeed() {
            state.speedMultiplier = (state.speedMultiplier % 8) + 1;
            const lbl = document.getElementById('speed-label');
            if (lbl) lbl.textContent = currentLang === 'ko' ? `${state.speedMultiplier}배속` : `${state.speedMultiplier}x Speed`;
        }

        // 목표 턴 분량 순환 (60턴 -> 80턴 -> 100턴)
        function cycleTargetTurns() {
            const options = [60, 80, 100];
            const idx = options.indexOf(state.maxTurn);
            state.maxTurn = options[(idx + 1) % options.length];
            if (state.turn > state.maxTurn) state.maxTurn = 100;
            document.getElementById('lbl-turn-goal').textContent = currentLang === 'ko' ? `대장정 ${state.maxTurn}턴` : `Campaign ${state.maxTurn}T`;
            refreshUI();
            if (state.currentEvent && !state.isOver) renderCurrentEventView();
        }

        // 50~100턴 절차적+수제 이벤트 생성 (KOR / EN 완벽 대응)
        function generateProceduralEvent(turnNum) {
            const regions = [
                { dir: 'korea', zoneKo: '한반도 중남부', zoneEn: 'Central & Southern Korea', descKo: '삼한 내륙의 성읍과 곡창지대', descEn: 'inland fortresses and granaries of the Korean Peninsula' },
                { dir: 'china', zoneKo: '중원 대륙과 황해안', zoneEn: 'China Mainland & Yellow Sea', descKo: '산둥반도와 황하·장강 일대의 대도시', descEn: 'Shandong Peninsula and major cities along the Yellow and Yangtze Rivers' },
                { dir: 'wa', zoneKo: '대한해협과 왜 열도', zoneEn: 'Korea Strait & Wa Archipelago', descKo: '대마도와 규슈·아스카의 해상 거점', descEn: 'maritime strongholds across Tsushima, Kyushu, and Asuka' },
                { dir: 'steppe', zoneKo: '만주 벌판과 북방 초원', zoneEn: 'Manchuria & Northern Steppe', descKo: '송화강과 흥안령 너머 유목 부족 영토', descEn: 'nomadic territories beyond the Songhua River and Khingan Range' }
            ];
            const reg = regions[turnNum % regions.length];
            const themes = [
                { preKo: '춘계 대원정 건의', preEn: 'Spring Grand Expedition', actKo: '정예 군단을 파견해 요충지를 공략', actEn: 'dispatch elite corps to capture key strongholds' },
                { preKo: '국경 무역권 분쟁', preEn: 'Border Trade Dispute', actKo: '무장 상단과 수군을 보내 상권을 제압', actEn: 'send armed fleets to dominate trade routes' },
                { preKo: '인접 세력의 도발', preEn: 'Neighboring Faction Provocation', actKo: '선제 타격으로 적의 전진 기지를 점령', actEn: 'launch a preemptive strike to seize forward bases' },
                { preKo: '제국 영역 확장 어전회의', preEn: 'Imperial Expansion Council', actKo: '신무기 부대를 앞세워 변경 영토를 개척', actEn: 'spearhead frontier expansion with new siege weapons' }
            ];
            const thm = themes[Math.floor(Math.random() * themes.length)];

            return {
                tag: `제${turnNum}턴 정무 · ${reg.zoneKo}`,
                tagEn: `Turn ${turnNum} Council · ${reg.zoneEn}`,
                title: `[${thm.preKo}] ${reg.zoneKo} 패권 장악 결단`,
                titleEn: `[${thm.preEn}] Struggle for Supremacy in ${reg.zoneEn}`,
                desc: `재위 ${turnNum}년차를 맞아 ${reg.descKo}에서 중대한 정세 변화가 일어났습니다. ${thm.actKo}하시겠습니까, 아니면 내치와 외교로 국력을 비축하시겠습니까?`,
                descEn: `In year ${turnNum} of your reign, a major shift has occurred across the ${reg.descEn}. Will you ${thm.actEn}, or focus on domestic growth and diplomacy?`,
                choices: [
                    {
                        text: `정예군을 출정시켜 ${reg.zoneKo}의 핵심 구역 두 곳을 무력으로 점령한다`,
                        textEn: `March elite forces to capture two key regions in ${reg.zoneEn} by force`,
                        baseProb: 56, statKey: 'military',
                        winEff: { economy: -4, military: -4, authority: +12, diplomacy: -2 },
                        failEff: { economy: -10, military: -12, authority: -8, diplomacy: -4 },
                        winTerr: { dir: reg.dir, gain: 2 },
                        failTerr: null,
                        winMsg: `${reg.zoneKo} 원정군이 적의 방어선을 격파하고 두 곳의 전략 구역을 새로 복속시켰습니다!`,
                        winMsgEn: `Our expedition crushed enemy defenses in ${reg.zoneEn} and annexed two strategic regions!`,
                        failMsg: `${reg.zoneKo}의 험준한 지형과 기상 악화로 원정군이 목표를 달성하지 못하고 귀환했습니다.`,
                        failMsgEn: `Harsh terrain and bad weather in ${reg.zoneEn} forced our expedition to return without gains.`
                    },
                    {
                        text: `외교 사절과 황금을 보내 ${reg.zoneKo} 거점 한 곳을 평화적으로 넘겨받는다`,
                        textEn: `Send envoys and gold to peacefully take over one stronghold in ${reg.zoneEn}`,
                        baseProb: 74, statKey: 'diplomacy',
                        winEff: { economy: -6, military: +4, authority: +8, diplomacy: +12 },
                        failEff: { economy: -10, military: -2, authority: -4, diplomacy: -8 },
                        winTerr: { dir: reg.dir, gain: 1 },
                        failTerr: null,
                        winMsg: `능숙한 외교 협상으로 피 한 방울 흘리지 않고 ${reg.zoneKo} 거점 한 곳을 편입했습니다!`,
                        winMsgEn: `Through deft diplomacy, we annexed a stronghold in ${reg.zoneEn} without bloodshed!`,
                        failMsg: `상대측 호족들이 약속을 번복하여 외교 자금만 소모하고 협상이 결렬되었습니다.`,
                        failMsgEn: `Local lords broke their promise, wasting our diplomatic funds as talks collapsed.`
                    },
                    {
                        text: `원정을 미루고 국내 농경지 개간과 병기창 확충에 집중하여 국력을 크게 높인다`,
                        textEn: `Postpone campaigns and focus on farmland reclamation and armories to boost power`,
                        baseProb: 89, statKey: 'economy',
                        winEff: { economy: +14, military: +10, authority: +10, diplomacy: +6 },
                        failEff: { economy: +3, military: +2, authority: -2, diplomacy: 0 },
                        winTerr: null,
                        failTerr: null,
                        winMsg: `내정 개혁과 둔전 개발이 큰 성과를 거두어 국고와 병력이 충실해졌습니다!`,
                        winMsgEn: `Domestic reforms and farmland expansion succeeded, filling our treasury and barracks!`,
                        failMsg: `일부 지방의 가뭄으로 세수 확보가 예상보다 소폭에 그쳤습니다.`,
                        failMsgEn: `Regional droughts kept tax revenue gains smaller than anticipated.`
                    },
                    {
                        text: `[대규모 총력전] 국력을 총동원해 ${reg.zoneKo} 세 구역을 단숨에 석권한다!`,
                        textEn: `[All-Out Campaign] Mobilize the entire realm to conquer three regions in ${reg.zoneEn}!`,
                        baseProb: 37, statKey: 'military',
                        winEff: { economy: +8, military: -8, authority: +20, diplomacy: -6 },
                        failEff: { economy: -14, military: -16, authority: -12, diplomacy: -8 },
                        winTerr: { dir: reg.dir, gain: 3 },
                        failTerr: { dir: reg.dir, gain: 0, lose: 1, enemy: 'china' },
                        winMsg: `천하를 뒤흔든 대승리! ${reg.zoneKo} 세 구역을 동시에 정벌하며 제국의 판도를 크게 넓혔습니다!`,
                        winMsgEn: `Earth-shaking victory! We conquered three regions in ${reg.zoneEn} at once!`,
                        failMsg: `무리한 대회전이 적의 집중 반격에 부딪혀 패배하고 변경 거점 한 곳을 상실했습니다.`,
                        failMsgEn: `Our massive offensive met a fierce counterattack, losing one border province in defeat.`
                    }
                ]
            };
        }

        function initNewRandomGame() {
            const keys = ['goguryeo', 'baekje', 'silla'];
            const kId = keys[Math.floor(Math.random() * keys.length)];
            const kInfo = KINGDOMS[kId];
            const rulerObj = kInfo.rulers[Math.floor(Math.random() * kInfo.rulers.length)];

            state.kingdomId = kId;
            state.rulerObj = rulerObj;
            state.year = kInfo.startYear;
            state.turn = 1;
            state.stats = { ...kInfo.stats };
            state.isOver = false;
            state.isPopupOpen = false;
            state.lastPopupData = null;
            state.timeLeft = state.maxTime;
            state.lastFrameTime = performance.now();

            state.provinces = {};
            PROVINCES_40.forEach(p => {
                state.provinces[p.id] = { ...p };
            });

            document.getElementById('result-popup-overlay').classList.add('hidden');
            document.getElementById('ending-box').classList.add('hidden');
            document.getElementById('choices-box').classList.remove('hidden');
            document.getElementById('timer-control-bar').classList.remove('hidden');

            buildMapSVG();
            refreshMapSVG();
            refreshUI();
            refillDeck();
            pickNextEvent();
        }

        function refillDeck() {
            state.deck = [...CORE_EVENTS].sort(() => Math.random() - 0.5);
        }

        function calcChoiceSuccessProb(ch) {
            const statVal = state.stats[ch.statKey] || 60;
            const bonus = Math.round((statVal - 50) * 0.35);
            return Math.max(15, Math.min(96, ch.baseProb + bonus));
        }

        /* =========================================================================
           6. 40분획 SVG 지도 렌더링 (KOR / EN 지명 표시)
           ========================================================================= */
        function buildMapSVG() {
            const provLayer = document.getElementById('provinces-layer');
            const labelLayer = document.getElementById('labels-layer');
            provLayer.innerHTML = '';
            labelLayer.innerHTML = '';

            PROVINCES_40.forEach(p => {
                const pathEl = document.createElementNS('http://www.w3.org/2000/svg', 'path');
                pathEl.setAttribute('id', `cell-${p.id}`);
                pathEl.setAttribute('d', p.path);
                pathEl.setAttribute('class', 'prov-cell');

                pathEl.addEventListener('mouseenter', () => {
                    const curOwner = state.provinces[p.id].owner;
                    const fMeta = FACTION_META[curOwner];
                    if (currentLang === 'ko') {
                        document.getElementById('map-hover-info').textContent = `[${p.zone}] ${p.name} — 현재 지배: ${fMeta.name}`;
                    } else {
                        document.getElementById('map-hover-info').textContent = `[${p.zoneEn}] ${p.nameEn} — Controlled by: ${fMeta.nameEn}`;
                    }
                });
                provLayer.appendChild(pathEl);

                const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                txt.setAttribute('x', p.label[0]);
                txt.setAttribute('y', p.label[1]);
                txt.setAttribute('text-anchor', 'middle');
                txt.setAttribute('dominant-baseline', 'central');
                txt.setAttribute('font-size', p.capital ? '9' : '7.5');
                txt.setAttribute('font-weight', p.capital ? '800' : '600');
                txt.setAttribute('fill', '#f8fafc');
                txt.setAttribute('style', 'text-shadow: 0 1px 2px #000, 0 0 4px #000;');
                const dispName = currentLang === 'ko' ? p.name : p.nameEn;
                txt.textContent = p.capital ? `★${dispName}` : dispName;
                labelLayer.appendChild(txt);
            });
        }

        function refreshMapSVG() {
            const counts = { goguryeo: 0, baekje: 0, silla: 0, china: 0, nomad: 0, wa: 0 };

            Object.values(state.provinces).forEach(p => {
                counts[p.owner] = (counts[p.owner] || 0) + 1;
                const el = document.getElementById(`cell-${p.id}`);
                if (!el) return;
                const isMine = (p.owner === state.kingdomId);
                el.setAttribute('fill', FACTION_META[p.owner].fill);
                el.setAttribute('fill-opacity', isMine ? '0.88' : '0.58');
            });

            Object.keys(counts).forEach(k => {
                const cEl = document.getElementById(`cnt-${k}`);
                if (cEl) cEl.textContent = counts[k];
            });

            const myCnt = counts[state.kingdomId] || 0;
            document.getElementById('hdr-terr-count').textContent = myCnt;
            document.getElementById('banner-terr-mobile').textContent = currentLang === 'ko'
                ? `영토 ${myCnt} / 40`
                : `Land ${myCnt} / 40`;
        }

        function getProvincesToGain(terrObj) {
            if (!terrObj || !terrObj.gain) return [];
            const myK = state.kingdomId;
            const res = [];
            const candidates = [...(terrObj.pick || []), ...(EXPANSION_PATHS[terrObj.dir] || EXPANSION_PATHS.any)];

            for (const pid of candidates) {
                if (res.length >= terrObj.gain) break;
                if (!res.includes(pid) && state.provinces[pid] && state.provinces[pid].owner !== myK) {
                    res.push(pid);
                }
            }
            for (const p of PROVINCES_40) {
                if (res.length >= terrObj.gain) break;
                if (!res.includes(p.id) && state.provinces[p.id].owner !== myK) {
                    res.push(p.id);
                }
            }
            return res;
        }

        function getProvincesToLose(terrObj) {
            if (!terrObj || !terrObj.lose) return [];
            const myK = state.kingdomId;
            const myProvs = Object.values(state.provinces).filter(p => p.owner === myK && p.capital !== myK);
            if (myProvs.length === 0) {
                const allMine = Object.values(state.provinces).filter(p => p.owner === myK);
                return allMine.slice(-terrObj.lose).map(p => p.id);
            }
            return myProvs.slice(-terrObj.lose).map(p => p.id);
        }

        function previewTerritoryOnMap(ch, show) {
            PROVINCES_40.forEach(p => {
                const el = document.getElementById(`cell-${p.id}`);
                if (el) el.classList.remove('prov-gain', 'prov-lose');
            });
            if (!show || !ch) return;

            if (ch.winTerr) {
                getProvincesToGain(ch.winTerr).forEach(pid => {
                    const el = document.getElementById(`cell-${pid}`);
                    if (el) el.classList.add('prov-gain');
                });
            }
            if (ch.failTerr) {
                getProvincesToLose(ch.failTerr).forEach(pid => {
                    const el = document.getElementById(`cell-${pid}`);
                    if (el) el.classList.add('prov-lose');
                });
            }
        }

        /* =========================================================================
           7. 정무 이벤트 표시 및 차등 확률 선택지 렌더링 (KOR / EN 지원)
           ========================================================================= */
        function pickNextEvent() {
            state.timeLeft = state.maxTime;
            state.lastFrameTime = performance.now();

            let ev;
            if (state.deck.length > 0 && state.turn % 2 === 1) {
                ev = state.deck.shift();
            } else {
                if (state.deck.length === 0) refillDeck();
                ev = (Math.random() < 0.65) ? state.deck.shift() : generateProceduralEvent(state.turn);
            }
            state.currentEvent = ev;
            renderCurrentEventView();
        }

        function renderCurrentEventView() {
            const ev = state.currentEvent;
            if (!ev) return;

            document.getElementById('ev-tag').textContent = currentLang === 'ko' ? ev.tag : (ev.tagEn || ev.tag);
            document.getElementById('ev-turn-info').textContent = currentLang === 'ko'
                ? `제${state.turn}턴 / 총 ${state.maxTurn}턴 (멸망 시 조기 종료)`
                : `Turn ${state.turn} / ${state.maxTurn} (Early end on national fall)`;
            document.getElementById('ev-title').textContent = currentLang === 'ko' ? ev.title : (ev.titleEn || ev.title);
            document.getElementById('ev-desc').textContent = currentLang === 'ko' ? ev.desc : (ev.descEn || ev.desc);

            const box = document.getElementById('choices-box');
            box.innerHTML = '';

            const statLabels = currentLang === 'ko'
                ? { economy: '국고', military: '군사', authority: '민심', diplomacy: '외교' }
                : { economy: 'Treasury', military: 'Military', authority: 'Support', diplomacy: 'Diplomacy' };

            ev.choices.slice(0, 4).forEach((ch, idx) => {
                const prob = calcChoiceSuccessProb(ch);
                let probBadgeCls = 'bg-emerald-500/20 text-emerald-300 border-emerald-400/40';
                if (prob < 48) probBadgeCls = 'bg-rose-500/20 text-rose-300 border-rose-400/40';
                else if (prob < 72) probBadgeCls = 'bg-amber-500/20 text-amber-300 border-amber-400/40';

                const btn = document.createElement('button');
                btn.className = 'w-full text-left p-2.5 sm:p-3 rounded-xl bg-white/5 hover:bg-white/10 border border-white/10 hover:border-amber-400/60 transition flex flex-col gap-1.5 group';

                const badges = [];
                if (ch.winTerr && ch.winTerr.gain > 0) {
                    const targets = getProvincesToGain(ch.winTerr).map(id => currentLang === 'ko' ? state.provinces[id].name : state.provinces[id].nameEn).join('·');
                    if (targets) {
                        const txt = currentLang === 'ko'
                            ? `성공 시 +영토 ${ch.winTerr.gain} (${targets})`
                            : `Win: +${ch.winTerr.gain} Land (${targets})`;
                        badges.push(`<span class="px-1.5 py-0.5 rounded border text-[10px] font-bold text-amber-200 bg-amber-500/20 border-amber-400/40">${txt}</span>`);
                    }
                }
                Object.entries(ch.winEff).forEach(([k, v]) => {
                    if (v > 0) {
                        badges.push(`<span class="px-1.5 py-0.5 rounded border text-[10px] font-semibold text-emerald-300 bg-emerald-500/10 border-emerald-500/30">${statLabels[k]} +${v}</span>`);
                    }
                });
                if (ch.failTerr && ch.failTerr.lose > 0) {
                    const rTxt = currentLang === 'ko' ? `실패 시 -영토 ${ch.failTerr.lose} 위험` : `Fail Risk: -${ch.failTerr.lose} Land`;
                    badges.push(`<span class="px-1.5 py-0.5 rounded border text-[10px] font-semibold text-rose-300 bg-rose-500/15 border-rose-400/30">${rTxt}</span>`);
                }

                const cText = currentLang === 'ko' ? ch.text : (ch.textEn || ch.text);
                const pText = currentLang === 'ko' ? `성공 ${prob}%` : `Win ${prob}%`;
                const checkText = currentLang === 'ko' ? `[${statLabels[ch.statKey]} 판정]` : `[${statLabels[ch.statKey]} Check]`;

                btn.innerHTML = `
                    <div class="flex items-start justify-between gap-2">
                        <span class="text-xs sm:text-sm font-semibold text-gray-100 group-hover:text-amber-300 transition leading-snug">${idx + 1}. ${cText}</span>
                        <span class="px-2 py-0.5 rounded border text-[11px] font-extrabold shrink-0 ${probBadgeCls}">
                            ${pText}
                        </span>
                    </div>
                    <div class="flex flex-wrap items-center gap-1">
                        <span class="text-[10px] text-gray-400 mr-1">${checkText}</span>
                        ${badges.join('')}
                    </div>
                `;

                btn.addEventListener('mouseenter', () => previewTerritoryOnMap(ch, true));
                btn.addEventListener('mouseleave', () => previewTerritoryOnMap(ch, false));
                btn.addEventListener('click', () => resolveChoiceWithPopup(ch, false));
                box.appendChild(btn);
            });
        }

        /* =========================================================================
           8. 확률 판정 및 결과/발생 이벤트 팝업 표시 (KOR / EN 지원)
           ========================================================================= */
        function resolveChoiceWithPopup(ch, isTimeoutAuto = false) {
            if (state.isOver || state.isPopupOpen) return;
            state.isPopupOpen = true;
            previewTerritoryOnMap(null, false);

            const prob = calcChoiceSuccessProb(ch);
            const roll = Math.floor(Math.random() * 100) + 1;
            const isSuccess = (roll <= prob);
            const isCritSuccess = isSuccess && (roll <= Math.floor(prob * 0.25));
            const isCritFail = !isSuccess && (roll >= 92);

            const appliedEff = { ...(isSuccess ? ch.winEff : ch.failEff) };
            if (isCritSuccess) {
                Object.keys(appliedEff).forEach(k => { if (appliedEff[k] > 0) appliedEff[k] += 4; });
            }

            const chainPool = isSuccess ? CHAIN_EVENTS.win : CHAIN_EVENTS.fail;
            const chainEv = chainPool[Math.floor(Math.random() * chainPool.length)];

            const totalEff = { economy: 0, military: 0, authority: 0, diplomacy: 0 };
            ['economy', 'military', 'authority', 'diplomacy'].forEach(k => {
                const delta = (appliedEff[k] || 0) + (chainEv.eff[k] || 0);
                totalEff[k] = delta;
                state.stats[k] = Math.max(0, Math.min(100, state.stats[k] + delta));
            });

            const gainedIds = [];
            const lostIds = [];
            if (isSuccess && ch.winTerr) {
                getProvincesToGain(ch.winTerr).forEach(pid => {
                    state.provinces[pid].owner = state.kingdomId;
                    gainedIds.push(pid);
                });
            } else if (!isSuccess && ch.failTerr) {
                getProvincesToLose(ch.failTerr).forEach(pid => {
                    state.provinces[pid].owner = ch.failTerr.enemy || 'china';
                    lostIds.push(pid);
                });
            }

            refreshMapSVG();
            refreshUI();

            state.lastPopupData = {
                ch, isTimeoutAuto, prob, roll, isSuccess, isCritSuccess, isCritFail,
                totalEff, gainedIds, lostIds, chainEv
            };
            renderPopupContent(state.lastPopupData);
            document.getElementById('result-popup-overlay').classList.remove('hidden');
        }

        function renderPopupContent(d) {
            const badgeEl = document.getElementById('popup-outcome-badge');
            const headerBar = document.getElementById('popup-header-bar');

            if (d.isCritSuccess) {
                badgeEl.textContent = currentLang === 'ko' ? '★ 대성공' : '★ GREAT SUCCESS';
                badgeEl.className = 'px-2.5 py-0.5 rounded-md text-xs font-extrabold border bg-amber-500/25 text-amber-200 border-amber-400';
                headerBar.className = 'px-5 py-3.5 border-b border-amber-500/30 bg-amber-500/10 flex items-center justify-between';
            } else if (d.isSuccess) {
                badgeEl.textContent = currentLang === 'ko' ? '성공' : 'SUCCESS';
                badgeEl.className = 'px-2.5 py-0.5 rounded-md text-xs font-extrabold border bg-emerald-500/25 text-emerald-200 border-emerald-400';
                headerBar.className = 'px-5 py-3.5 border-b border-emerald-500/30 bg-emerald-500/10 flex items-center justify-between';
            } else if (d.isCritFail) {
                badgeEl.textContent = currentLang === 'ko' ? '▲ 참패' : '▲ CRITICAL DEFEAT';
                badgeEl.className = 'px-2.5 py-0.5 rounded-md text-xs font-extrabold border bg-rose-600/30 text-rose-200 border-rose-400';
                headerBar.className = 'px-5 py-3.5 border-b border-rose-500/30 bg-rose-500/15 flex items-center justify-between';
            } else {
                badgeEl.textContent = currentLang === 'ko' ? '실패' : 'FAILURE';
                badgeEl.className = 'px-2.5 py-0.5 rounded-md text-xs font-extrabold border bg-orange-500/25 text-orange-200 border-orange-400';
                headerBar.className = 'px-5 py-3.5 border-b border-orange-500/30 bg-orange-500/10 flex items-center justify-between';
            }

            if (currentLang === 'ko') {
                document.getElementById('popup-prob-detail').textContent = d.isTimeoutAuto
                    ? `제한 시간 60초 경과 자동 결단 (성공 확률 ${d.prob}% 판정)`
                    : `성공 확률 ${d.prob}% (판정값 ${d.roll})`;
                document.getElementById('popup-turn-label').textContent = `서기 ${state.year}년 (제${state.turn}턴)`;
                document.getElementById('popup-main-title').textContent = d.isTimeoutAuto
                    ? `[시간 초과 긴급 재가] ${d.ch.text}`
                    : d.ch.text;
                document.getElementById('popup-main-desc').textContent = d.isSuccess ? d.ch.winMsg : d.ch.failMsg;
            } else {
                const cTxt = d.ch.textEn || d.ch.text;
                document.getElementById('popup-prob-detail').textContent = d.isTimeoutAuto
                    ? `60s Timeout Auto-Decision (Success Rate ${d.prob}%)`
                    : `Success Rate ${d.prob}% (Roll ${d.roll})`;
                document.getElementById('popup-turn-label').textContent = `AD ${state.year} (Turn ${state.turn})`;
                document.getElementById('popup-main-title').textContent = d.isTimeoutAuto
                    ? `[Timeout Emergency Decree] ${cTxt}`
                    : cTxt;
                document.getElementById('popup-main-desc').textContent = d.isSuccess
                    ? (d.ch.winMsgEn || d.ch.winMsg)
                    : (d.ch.failMsgEn || d.ch.failMsg);
            }

            const statLabels = currentLang === 'ko'
                ? { economy: '국고', military: '군사', authority: '민심', diplomacy: '외교' }
                : { economy: 'Treasury', military: 'Military', authority: 'Support', diplomacy: 'Diplomacy' };

            const statChips = [];
            Object.entries(d.totalEff).forEach(([k, v]) => {
                if (v !== 0) {
                    const cls = v > 0
                        ? 'text-emerald-300 bg-emerald-500/15 border-emerald-500/30'
                        : 'text-rose-300 bg-rose-500/15 border-rose-500/30';
                    statChips.push(`<span class="px-2 py-0.5 rounded border text-xs font-bold ${cls}">${statLabels[k]} ${v > 0 ? '+' : ''}${v}</span>`);
                }
            });
            document.getElementById('popup-stat-chips').innerHTML = statChips.join('') || (currentLang === 'ko'
                ? '<span class="text-xs text-gray-400">국력 수치 변동 없음</span>'
                : '<span class="text-xs text-gray-400">No change in national stats</span>');

            const terrEl = document.getElementById('popup-terr-change');
            if (d.gainedIds.length > 0) {
                const names = d.gainedIds.map(id => currentLang === 'ko' ? state.provinces[id].name : state.provinces[id].nameEn).join(', ');
                terrEl.textContent = currentLang === 'ko'
                    ? `★ 신규 점령 영토 (+${d.gainedIds.length}분획): ${names}`
                    : `★ Captured Territories (+${d.gainedIds.length}): ${names}`;
                terrEl.className = 'text-xs font-bold text-amber-300';
            } else if (d.lostIds.length > 0) {
                const names = d.lostIds.map(id => currentLang === 'ko' ? state.provinces[id].name : state.provinces[id].nameEn).join(', ');
                terrEl.textContent = currentLang === 'ko'
                    ? `▲ 상실한 변경 영토 (-${d.lostIds.length}분획): ${names}`
                    : `▲ Lost Border Territories (-${d.lostIds.length}): ${names}`;
                terrEl.className = 'text-xs font-bold text-rose-400';
            } else {
                terrEl.textContent = currentLang === 'ko' ? '이번 턴에는 영토 경계 변동이 없습니다.' : 'No territorial border changes this turn.';
                terrEl.className = 'text-xs text-gray-400';
            }

            document.getElementById('popup-chain-tag').textContent = currentLang === 'ko'
                ? `파생 발생 이벤트 · ${d.chainEv.tag}`
                : `Follow-up Event · ${d.chainEv.tagEn || d.chainEv.tag}`;
            document.getElementById('popup-chain-title').textContent = currentLang === 'ko'
                ? d.chainEv.title
                : (d.chainEv.titleEn || d.chainEv.title);
            document.getElementById('popup-chain-desc').textContent = currentLang === 'ko'
                ? d.chainEv.desc
                : (d.chainEv.descEn || d.chainEv.desc);
            const chainEffStr = Object.entries(d.chainEv.eff).map(([k, v]) => `${statLabels[k]} ${v > 0 ? '+' : ''}${v}`).join(', ');
            document.getElementById('popup-chain-eff').textContent = chainEffStr;
            document.getElementById('popup-chain-eff').className = d.isSuccess ? 'text-[11px] font-bold text-emerald-300' : 'text-[11px] font-bold text-rose-300';
        }

        function closeResultPopupAndAdvance() {
            document.getElementById('result-popup-overlay').classList.add('hidden');
            state.isPopupOpen = false;

            const myCnt = Object.values(state.provinces).filter(p => p.owner === state.kingdomId).length;
            const s = state.stats;

            // 1) 국가 멸망 시에만 조기 종료
            if (s.economy <= 0 || s.military <= 0 || s.authority <= 0 || s.diplomacy <= 0 || myCnt <= 0) {
                let failReason = currentLang === 'ko' ? '국력 고갈로 인한 왕조의 멸망' : 'Dynasty Fall due to National Exhaustion';
                if (s.economy <= 0) failReason = currentLang === 'ko' ? '국고 파탄으로 인한 군대 해산과 왕조 멸망' : 'National Bankruptcy and Dissolution of the Army';
                else if (s.military <= 0) failReason = currentLang === 'ko' ? '군사력 붕괴와 외세의 도성 함락으로 국가 멸망' : 'Military Collapse and Fall of the Royal Capital';
                else if (s.authority <= 0) failReason = currentLang === 'ko' ? '민심 이반과 전국적인 반란으로 왕조 전복' : 'Nationwide Rebellion and Overthrow of the Crown';
                else if (s.diplomacy <= 0) failReason = currentLang === 'ko' ? '외교적 완전 고립과 열강 연합군의 총공세로 국가 멸망' : 'Total Diplomatic Isolation and Coalition Invasion';
                else if (myCnt <= 0) failReason = currentLang === 'ko' ? '모든 영토 상실로 인한 왕조 멸망' : 'Loss of All Territories and Fall of the Kingdom';

                const failDesc = currentLang === 'ko'
                    ? `제${state.turn}턴(서기 ${state.year}년), 국가의 핵심 기반이 무너지며 대장정을 완주하지 못하고 역사의 뒤안길로 사라졌습니다.`
                    : `On Turn ${state.turn} (AD ${state.year}), the foundations of the realm collapsed before completing the grand campaign.`;

                showEnding(false, failReason, failDesc, myCnt);
                return;
            }

            // 2) 목표 턴(50~100턴) 완주 결산
            if (state.turn >= state.maxTurn) {
                evaluateFullCampaignEnding(myCnt);
                return;
            }

            state.turn += 1;
            state.year += 2;
            refreshUI();
            pickNextEvent();
        }

        function evaluateFullCampaignEnding(myCnt) {
            const kInfo = KINGDOMS[state.kingdomId];
            const kName = currentLang === 'ko' ? kInfo.name : kInfo.nameEn;
            const myProvs = Object.values(state.provinces).filter(p => p.owner === state.kingdomId);
            const chinaCnt = myProvs.filter(p => p.id.startsWith('c')).length;
            const waCnt = myProvs.filter(p => p.id.startsWith('w')).length;
            const steppeCnt = myProvs.filter(p => p.id.startsWith('m')).length;

            if (myCnt >= 32) {
                showEnding(
                    true,
                    currentLang === 'ko' ? `동아시아 천하를 완전히 통일한 ${kName} 대제국` : `Supreme ${kName} Empire Unifying East Asia`,
                    currentLang === 'ko'
                        ? `총 ${state.maxTurn}턴의 대장정 끝에 한반도와 만주, 중원 대륙, 왜 열도까지 40분획 중 ${myCnt}개 구역을 제패하며 초거대 제국을 완성하셨습니다!`
                        : `After ${state.maxTurn} turns, you conquered ${myCnt} of 40 regions across Korea, Manchuria, China, and Japan, forging an unprecedented super-empire!`,
                    myCnt
                );
            } else if (chinaCnt >= 6) {
                showEnding(
                    true,
                    currentLang === 'ko' ? `중원 대륙을 정벌한 동방의 천자국` : `Conqueror of the Chinese Mainland`,
                    currentLang === 'ko'
                        ? `황하와 양쯔강을 넘어 장안과 낙양, 강남 등 중원 대륙 ${chinaCnt}개 구역을 복속시키고 대륙의 새로운 지배자로 군림하셨습니다!`
                        : `Crossing the Yellow and Yangtze Rivers, you subdued ${chinaCnt} continental regions and reigned as the supreme ruler of East Asia!`,
                    myCnt
                );
            } else if (waCnt >= 5) {
                showEnding(
                    true,
                    currentLang === 'ko' ? `왜 열도와 바닷길을 제패한 대해양 제국` : `Grand Maritime Empire of the Archipelago`,
                    currentLang === 'ko'
                        ? `대한해협을 건너 규슈와 아스카 기나이, 간토까지 왜 열도 ${waCnt}개 구역에 담로를 설치하고 동아시아 해양 패권을 장악하셨습니다!`
                        : `Across the Korea Strait, you established royal provinces over ${waCnt} archipelago regions and dominated the seas!`,
                    myCnt
                );
            } else if (steppeCnt >= 6) {
                showEnding(
                    true,
                    currentLang === 'ko' ? `만주와 몽골 대초원을 아우른 북방 대가한 제국` : `Supreme Khaganate of Manchuria and the Steppe`,
                    currentLang === 'ko'
                        ? `흥안령과 송화강, 몽골 초원까지 북방 ${steppeCnt}개 구역을 복속시켜 천하무적의 기마 제국을 건설하셨습니다!`
                        : `Subduing ${steppeCnt} northern regions up to the Mongol Steppe, you built an invincible cavalry empire!`,
                    myCnt
                );
            } else {
                showEnding(
                    true,
                    currentLang === 'ko' ? `${state.maxTurn}턴 대장정을 완주한 ${kName}의 태평성대` : `Golden Age of ${kName} Completing ${state.maxTurn} Turns`,
                    currentLang === 'ko'
                        ? `수많은 외세의 침략과 시련을 이겨내고 총 ${myCnt}개 전략 구역을 굳건히 수호하며 찬란한 문화와 부국강병을 이룩하셨습니다.`
                        : `Overcoming countless invasions and trials, you safeguarded ${myCnt} strategic regions and achieved lasting prosperity.`,
                    myCnt
                );
            }
        }

        function showEnding(isWin, title, desc, myCnt) {
            state.isOver = true;
            document.getElementById('choices-box').classList.add('hidden');
            document.getElementById('timer-control-bar').classList.add('hidden');
            const endBox = document.getElementById('ending-box');
            endBox.classList.remove('hidden');

            document.getElementById('end-badge').textContent = isWin
                ? (currentLang === 'ko' ? `★ ${state.maxTurn}턴 대장정 완주 성취` : `★ ${state.maxTurn}-Turn Grand Campaign Completed`)
                : (currentLang === 'ko' ? `▲ 제${state.turn}턴 국가 멸망 (조기 종료)` : `▲ Turn ${state.turn} National Fall (Early Game Over)`);
            document.getElementById('end-title').textContent = title;
            document.getElementById('end-desc').textContent = desc;
            document.getElementById('end-summary').textContent = currentLang === 'ko'
                ? `최종 기록: 재위 ${state.turn}턴 (서기 ${state.year}년) | 최종 영토: ${myCnt} / 40분획 | 국고 ${state.stats.economy} · 군사 ${state.stats.military} · 민심 ${state.stats.authority} · 외교 ${state.stats.diplomacy}`
                : `Final Record: Turn ${state.turn} (AD ${state.year}) | Territories: ${myCnt} / 40 | Treasury ${state.stats.economy} · Military ${state.stats.military} · Support ${state.stats.authority} · Diplomacy ${state.stats.diplomacy}`;
        }

        function refreshUI() {
            const kInfo = KINGDOMS[state.kingdomId];
            const hdrK = document.getElementById('hdr-kingdom');
            hdrK.textContent = currentLang === 'ko' ? kInfo.name : kInfo.nameEn;
            hdrK.className = `px-2.5 py-0.5 rounded text-xs font-extrabold border shrink-0 ${kInfo.badgeCls}`;

            document.getElementById('hdr-ruler').textContent = currentLang === 'ko' ? state.rulerObj.ko : state.rulerObj.en;
            document.getElementById('hdr-year').textContent = currentLang === 'ko'
                ? `서기 ${state.year}년 (${state.turn} / ${state.maxTurn}턴)`
                : `AD ${state.year} (${state.turn} / ${state.maxTurn}T)`;

            ['economy', 'military', 'authority', 'diplomacy'].forEach(k => {
                const v = state.stats[k];
                document.getElementById(`val-${k}`).textContent = v;
                document.getElementById(`bar-${k}`).style.width = `${v}%`;
            });

            const myCnt = Object.values(state.provinces).filter(p => p.owner === state.kingdomId).length;
            const moodEl = document.getElementById('banner-mood');
            const descEl = document.getElementById('banner-desc');
            const kDisp = currentLang === 'ko' ? kInfo.name : kInfo.nameEn;

            if (Object.values(state.stats).some(v => v <= 20)) {
                moodEl.textContent = currentLang === 'ko' ? '국가 멸망 위기 경보' : 'National Crisis Alert';
                moodEl.className = 'px-2 py-0.5 rounded bg-red-950/90 border border-red-500/40 text-[11px] font-bold text-red-300';
                descEl.textContent = currentLang === 'ko'
                    ? '국정 지표가 0이 되면 국가가 멸망하여 조기 종료됩니다! 내정을 회복하십시오.'
                    : 'If any stat hits 0, the kingdom falls! Restore stability immediately.';
            } else if (myCnt >= 18) {
                moodEl.textContent = currentLang === 'ko' ? `${kDisp} 대제국 원정군 행군 중` : `${kDisp} Grand Imperial Army Marching`;
                moodEl.className = 'px-2 py-0.5 rounded bg-amber-950/90 border border-amber-500/40 text-[11px] font-bold text-amber-300';
                descEl.textContent = currentLang === 'ko'
                    ? '대륙과 열도의 성읍들이 잇달아 항복하며 제국의 위용이 하늘을 찌릅니다!'
                    : 'Fortresses across the continent and archipelago surrender before our mighty empire!';
            } else {
                moodEl.textContent = currentLang === 'ko' ? `${kDisp} 도성 안정 · 정벌 준비` : `${kDisp} Capital Stable · Ready to March`;
                moodEl.className = 'px-2 py-0.5 rounded bg-black/75 border border-white/15 text-[11px] font-bold text-emerald-300';
                descEl.textContent = currentLang === 'ko'
                    ? '군사들이 병기를 갈고닦으며 어명이 떨어지기만을 기다리고 있습니다.'
                    : 'Soldiers sharpen their blades and await Your Majesty’s royal command.';
            }
        }

        /* =========================================================================
           9. 첫 화면 10종 도트 캐릭터 일러스트 렌더러 & 인게임 도트 배너 루프
           ========================================================================= */
        // 도트 캐릭터 헬퍼 함수 (왕, 장군, 기병, 궁수, 전함 등 픽셀 스프라이트 드로잉)
        function drawPixelCharacter(ctx, x, y, helmColor, armorColor, weaponType, tickOffset = 0) {
            const bob = Math.floor(Math.sin((bannerTick * 0.12) + tickOffset) * 1.5);
            const py = y + bob;
            // 투구 & 깃털 장식
            ctx.fillStyle = '#fbbf24';
            ctx.fillRect(x + 2, py - 14, 2, 3);
            ctx.fillStyle = helmColor;
            ctx.fillRect(x, py - 11, 6, 3);
            // 얼굴
            ctx.fillStyle = '#fde68a';
            ctx.fillRect(x + 1, py - 8, 4, 3);
            // 갑옷 몸통
            ctx.fillStyle = armorColor;
            ctx.fillRect(x, py - 5, 6, 6);
            // 다리
            ctx.fillStyle = '#1e293b';
            ctx.fillRect(x + 1, py + 1, 2, 3);
            ctx.fillRect(x + 3, py + 1, 2, 3);
            // 무기
            ctx.fillStyle = '#e2e8f0';
            if (weaponType === 'spear') {
                ctx.fillRect(x + 7, py - 15, 1, 18);
                ctx.fillStyle = helmColor;
                ctx.fillRect(x + 8, py - 13, 3, 2);
            } else if (weaponType === 'sword') {
                ctx.fillRect(x + 6, py - 10, 2, 7);
            } else if (weaponType === 'bow') {
                ctx.fillStyle = '#d97706';
                ctx.fillRect(x + 6, py - 10, 2, 8);
            }
        }

        function renderIntroPixelCanvas() {
            const canvas = document.getElementById('intro-pixel-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            const W = canvas.width;
            const H = canvas.height;
            const sc = INTRO_PIXEL_SCENES[selectedIntroSceneIndex] || INTRO_PIXEL_SCENES[0];

            // 1. 하늘 그라데이션 (도트 단차 스타일)
            ctx.fillStyle = sc.skyTop;
            ctx.fillRect(0, 0, W, Math.floor(H * 0.45));
            ctx.fillStyle = sc.skyBot;
            ctx.fillRect(0, Math.floor(H * 0.45), W, Math.floor(H * 0.35));

            // 별빛 / 불티 도트
            for (let i = 0; i < 28; i++) {
                const sx = (i * 37 + Math.floor(bannerTick * 0.2)) % W;
                const sy = (i * 19) % 95;
                ctx.fillStyle = (i % 3 === 0) ? sc.accent : '#ffffff';
                ctx.fillRect(sx, sy, 2, 2);
            }

            // 해 또는 보름달
            ctx.fillStyle = sc.accent;
            ctx.fillRect(250, 22, 24, 24);
            ctx.fillStyle = '#fef3c7';
            ctx.fillRect(254, 26, 16, 16);

            // 원경 산맥 실루엣 (한반도 산세)
            ctx.fillStyle = '#0f172a';
            for (let x = 0; x < W; x += 4) {
                const mh = Math.floor(Math.sin(x * 0.03 + selectedIntroSceneIndex) * 18 + Math.cos(x * 0.08) * 8 + 34);
                ctx.fillRect(x, 132 - mh, 4, mh);
            }

            // 지면 또는 바다
            ctx.fillStyle = sc.ground;
            ctx.fillRect(0, 132, W, H - 132);

            // 10개 장면별 고유 도트 일러스트 연출
            const id = sc.id;
            if (id === 1 || id === 5 || id === 8) {
                // 고구려 테마: 성곽 + 개마무사 기병대 돌격
                ctx.fillStyle = '#334155';
                ctx.fillRect(18, 88, 75, 44);
                for (let cx = 18; cx < 93; cx += 10) ctx.fillRect(cx, 82, 6, 6);
                // 삼족오 붉은 대군기
                ctx.fillStyle = '#dc2626';
                ctx.fillRect(46, 58, 22, 14);
                ctx.fillStyle = '#fbbf24';
                ctx.fillRect(53, 62, 6, 6);
                // 돌격하는 개마무사 도트 캐릭터 6기
                for (let i = 0; i < 6; i++) {
                    const hx = 95 + ((i * 36 + Math.floor(bannerTick * 0.7)) % 210);
                    const hy = 126 + (i % 2) * 10;
                    // 철갑마
                    ctx.fillStyle = '#475569';
                    ctx.fillRect(hx - 4, hy - 2, 14, 6);
                    ctx.fillRect(hx - 4, hy + 4, 2, 4);
                    ctx.fillRect(hx + 8, hy + 4, 2, 4);
                    drawPixelCharacter(ctx, hx, hy - 2, '#dc2626', '#991b1b', 'spear', i);
                }
            } else if (id === 2 || id === 6 || id === 9) {
                // 백제 테마: 황포돛배 선단 & 금동대향로 왕궁 캐릭터
                ctx.fillStyle = '#78350f';
                ctx.fillRect(20, 86, 86, 46);
                ctx.fillStyle = '#d97706';
                ctx.fillRect(14, 78, 98, 10);
                // 백제 어라하 & 박사 도트 캐릭터
                drawPixelCharacter(ctx, 45, 128, '#fbbf24', '#d97706', 'sword', 1);
                drawPixelCharacter(ctx, 62, 128, '#f59e0b', '#b45309', 'spear', 2);
                drawPixelCharacter(ctx, 78, 128, '#38bdf8', '#0284c7', 'sword', 3);
                // 황포돛배 3척 항해
                for (let s = 0; s < 3; s++) {
                    const sx = 135 + ((s * 62 + Math.floor(bannerTick * 0.45)) % 175);
                    const sy = 130 + s * 11;
                    ctx.fillStyle = '#78350f';
                    ctx.fillRect(sx, sy, 32, 8);
                    ctx.fillStyle = '#f59e0b';
                    ctx.fillRect(sx + 8, sy - 20, 16, 20);
                    drawPixelCharacter(ctx, sx + 12, sy - 1, '#fbbf24', '#d97706', 'spear', s);
                }
            } else if (id === 3 || id === 7) {
                // 신라 테마: 첨성대 & 서라벌 신림과 화랑도 궁수 캐릭터들
                // 첨성대 도트 구조물
                ctx.fillStyle = '#94a3b8';
                ctx.fillRect(46, 102, 22, 30);
                ctx.fillRect(50, 86, 14, 16);
                ctx.fillRect(48, 82, 18, 4);
                // 신라 여왕/마립간과 화랑도 대열
                drawPixelCharacter(ctx, 88, 130, '#fbbf24', '#059669', 'sword', 0);
                for (let h = 0; h < 6; h++) {
                    const hx = 118 + h * 30;
                    const hy = 132 + (h % 2) * 8;
                    drawPixelCharacter(ctx, hx, hy, '#10b981', '#047857', 'bow', h);
                }
            } else {
                // 삼국 군주 한강 회동 / 백두산 천지 대통일 (#4, #10)
                // 중앙 제단/천지 호수
                ctx.fillStyle = '#0284c7';
                ctx.fillRect(90, 136, 140, 18);
                // 고구려 태왕(좌), 백제 어라하(중), 신라 마립간(우)와 호위 무사들
                drawPixelCharacter(ctx, 75, 128, '#ef4444', '#991b1b', 'spear', 1);
                drawPixelCharacter(ctx, 92, 126, '#fbbf24', '#dc2626', 'sword', 2);

                drawPixelCharacter(ctx, 152, 124, '#fbbf24', '#d97706', 'sword', 3);
                drawPixelCharacter(ctx, 168, 128, '#f59e0b', '#92400e', 'spear', 4);

                drawPixelCharacter(ctx, 222, 126, '#fbbf24', '#059669', 'sword', 5);
                drawPixelCharacter(ctx, 238, 128, '#10b981', '#065f46', 'bow', 6);
            }
        }

        function mainLoop(now) {
            const dt = (now - state.lastFrameTime) / 1000;
            state.lastFrameTime = now;

            bannerTick += (0.8 + state.speedMultiplier * 0.2);

            // 첫 화면(인트로)이 열려 있을 때는 10종 도트 일러스트 캔버스 애니메이션만 구동
            if (isIntroActive) {
                renderIntroPixelCanvas();
                requestAnimationFrame(mainLoop);
                return;
            }

            // 본 게임 진행 중이고 팝업이 닫혀 있을 때만 60초 타이머 작동
            if (!state.isOver && !state.isPopupOpen && state.currentEvent) {
                state.timeLeft -= dt * state.speedMultiplier;
                if (state.timeLeft <= 0) {
                    state.timeLeft = 0;
                    const fallbackChoice = state.currentEvent.choices[2] || state.currentEvent.choices[0];
                    resolveChoiceWithPopup(fallbackChoice, true);
                }

                const secText = document.getElementById('timer-sec-text');
                const barEl = document.getElementById('timer-progress-bar');
                if (secText && barEl) {
                    const unit = currentLang === 'ko' ? '초' : 's';
                    secText.textContent = `${Math.max(0, state.timeLeft).toFixed(1)}${unit}`;
                    const pct = Math.max(0, Math.min(100, (state.timeLeft / state.maxTime) * 100));
                    barEl.style.width = `${pct}%`;
                }
            }

            // 인게임 상단 도트 상태 배너 애니메이션
            const canvas = document.getElementById('dot-banner');
            if (canvas) {
                const ctx = canvas.getContext('2d');
                const W = canvas.width;
                const H = canvas.height;
                const s = state.stats;
                const myCnt = Object.values(state.provinces).filter(p => p.owner === state.kingdomId).length;
                const isCrisis = Object.values(s).some(v => v <= 20);

                const colors = {
                    goguryeo: '#ef4444',
                    baekje: '#f59e0b',
                    silla: '#10b981'
                };
                const flagColor = colors[state.kingdomId];

                ctx.fillStyle = isCrisis ? '#3b0a10' : (myCnt >= 18 ? '#1e1b4b' : '#0f172a');
                ctx.fillRect(0, 0, W, H);

                ctx.fillStyle = isCrisis ? '#f43f5e' : '#fbbf24';
                ctx.fillRect(295, 10, 12, 12);

                ctx.fillStyle = '#1e293b';
                for (let x = 0; x < W; x += 4) {
                    const mh = Math.floor(Math.sin(x * 0.04) * 6 + 16);
                    ctx.fillRect(x, 52 - mh, 4, mh);
                }

                ctx.fillStyle = '#0369a1';
                ctx.fillRect(240, 52, 100, 16);
                const shipX = 250 + Math.floor((bannerTick * 0.3) % 65);
                ctx.fillStyle = '#78350f';
                ctx.fillRect(shipX, 47, 16, 5);
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(shipX + 4, 38, 7, 9);

                ctx.fillStyle = '#1c1917';
                ctx.fillRect(0, 52, 240, 16);
                ctx.fillStyle = '#78350f';
                ctx.fillRect(110, 34, 44, 18);
                ctx.fillStyle = flagColor;
                ctx.fillRect(104, 28, 56, 6);
                if (myCnt >= 12) {
                    ctx.fillStyle = '#92400e';
                    ctx.fillRect(118, 20, 28, 8);
                    ctx.fillStyle = '#fbbf24';
                    ctx.fillRect(114, 16, 36, 4);
                }

                const troops = Math.max(1, Math.min(7, Math.floor(s.military / 14)));
                for (let i = 0; i < troops; i++) {
                    const tx = 165 + Math.floor((bannerTick * 0.4 + i * 11) % 68);
                    ctx.fillStyle = flagColor;
                    ctx.fillRect(tx, 46, 3, 6);
                    ctx.fillStyle = '#e2e8f0';
                    ctx.fillRect(tx + 3, 42, 1, 10);
                }

                const folks = Math.max(1, Math.min(6, Math.floor(s.authority / 16)));
                for (let i = 0; i < folks; i++) {
                    const fx = 12 + Math.floor((bannerTick * 0.25 + i * 15) % 88);
                    ctx.fillStyle = '#38bdf8';
                    ctx.fillRect(fx, 47, 3, 5);
                }
            }

            requestAnimationFrame(mainLoop);
        }

        window.addEventListener('DOMContentLoaded', () => {
            pickRandomIntroScene();
            initVisitorCounter();
            initNewRandomGame();
            setLanguage('ko'); // 기본 언어 KOR 설정
            requestAnimationFrame(mainLoop);
        });
    </script>
</body>
</html>
