graph TD
    %% 그룹 정의
    subgraph Edge_Sensors [1. 엣지 노드 및 센서 (입력부)]
        HA_App[스마트 가전\n냉장고, 세탁기 등]
        mmWave[mmWave 레이더 센서]
        PIR[PIR 모션 센서]
        Cam[V4L2 지원 카메라]
        Door[도어 센서]
    end

    subgraph Main_Hub [2. 메인 컨트롤 허브 (Raspberry Pi 4)]
        HA_Core[Home Assistant Core]
        FastAPI_GW[FastAPI 게이트웨이]
        
        subgraph AI_Logic [AI 및 판단 로직]
            Algo1[가전 이상행동 분석\nIsolation Forest]
            Algo2[배회 판단 로직\nKalman Filter & 분산]
            Algo3[조명 연산 로직]
        end
        
        TTS_Engine[온디바이스 TTS\nPiper Engine / .onnx]
    end

    subgraph Actuators [3. 제어 및 출력 (액추에이터)]
        SmartLight[스마트 조명\nPWM/색온도 제어]
        TV[거실 TV\nAndroid TV]
        Soundbar[4W 사운드바\n유선 직결]
    end

    subgraph Cloud_API [4. 외부 클라우드 및 API]
        WeatherAPI[OpenWeatherMap API]
        GPU_Server[GPU 클라우드 서버\nRTX 3060]
        VisionAI[YOLOv8 고정밀 객체 탐지]
        VITS[VITS 음성 딥러닝 학습]
    end

    subgraph User_UI [5. 사용자 인터페이스]
        FlutterApp[Flutter 모바일 앱\n보호자용 스마트폰]
    end

    %% 데이터 흐름 연결
    HA_App -- 상태 데이터\nWebSocket --> HA_Core
    mmWave -- 점군 데이터 --> ESP[ESP32 MCU]
    ESP -- MQTT 통신 --> FastAPI_GW
    PIR & Door -- 이벤트 트리거 --> RPiZero[Raspberry Pi Zero]
    Cam -- JPEG 스냅샷 --> RPiZero
    RPiZero -- 무선 전송 --> FastAPI_GW

    HA_Core --> Algo1
    FastAPI_GW --> Algo2
    WeatherAPI -- 실시간 날씨/일몰 --> Algo3
    Algo3 --> SmartLight

    FastAPI_GW -- 현관 스냅샷 --> GPU_Server
    GPU_Server --> VisionAI
    VisionAI -- 검증 결과/이미지 --> FlutterApp

    FlutterApp -- 음성 녹음.wav\nRESTful API --> FastAPI_GW
    FastAPI_GW -- 음성 소스 --> GPU_Server
    GPU_Server --> VITS
    VITS -- 경량화 모델.onnx --> TTS_Engine
    
    Algo1 & Algo2 -- 위험 감지 트리거 --> HA_Core
    HA_Core -- 켜기/재생 제어 --> TV
    HA_Core -- 스크립트 전달 --> TTS_Engine
    TTS_Engine -- 실시간 음성 출력 --> Soundbar
    
    FastAPI_GW -- 긴급 푸시 알림 --> FlutterApp
