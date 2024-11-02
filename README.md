#include <windows.h>
#include <CommCtrl.h>

#pragma comment(lib, "Comctl32.lib")

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK SettingsProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK GameSelectProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK StoryProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

// 버튼 ID 정의
#define BUTTON_START 1
#define BUTTON_SETTINGS 2
#define BUTTON_EXIT 3
#define BUTTON_BACK 4
#define BUTTON_GAME_START 5
#define BUTTON_GAME_SELECT 6
#define CHECKBOX_FULLSCREEN 7
#define SLIDER_DIALOGUE_SPEED 8
#define SLIDER_MOVEMENT_SPEED 9
#define SLIDER_SOUND_VOLUME 10
#define BUTTON_START_JOYOONGJEON 11
#define BUTTON_STAGE_1 12 // 대흥산 전투 버튼
#define BUTTON_STAGE_2 13 // 개발 중 버튼
#define BUTTON_STAGE_3 14 // 개발 중 버튼
#define BUTTON_NEXT 15 // 다음 화면 버튼


// 설정 화면 및 게임 선택 화면 핸들
HWND hSettingsWnd = NULL;
HWND hGameSelectWnd = NULL;
HWND hStoryWnd = NULL;
HWND hStageSelectWnd = NULL; // 스테이지 선택 화면 핸들 추가

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK SettingsProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK GameSelectProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
LRESULT CALLBACK StageSelectProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam); // 함수 선언 추가

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE, LPSTR, int nCmdShow) {
    InitCommonControls();

    const wchar_t CLASS_NAME[] = L"MainWindowClass";
    WNDCLASS wc = {};

    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;

    RegisterClass(&wc);

    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        L"삼국지 조융전",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 500, 400,
        NULL,
        NULL,
        hInstance,
        NULL
    );

    if (hwnd == NULL) {
        return 0;
    }

    ShowWindow(hwnd, nCmdShow);

    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 게임 시작 버튼
        CreateWindow(
            L"BUTTON", L"게임 시작",
            WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            180, 100, 140, 40,
            hwnd, (HMENU)BUTTON_START, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);

        // 설정 버튼
        CreateWindow(
            L"BUTTON", L"설정",
            WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            180, 160, 140, 40,
            hwnd, (HMENU)BUTTON_SETTINGS, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);

        // 게임 종료 버튼
        CreateWindow(
            L"BUTTON", L"게임 종료",
            WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            180, 220, 140, 40,
            hwnd, (HMENU)BUTTON_EXIT, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        break;
    }
    case WM_COMMAND: {
        int wmId = LOWORD(wParam);
        switch (wmId) {
        case BUTTON_START: {
            const wchar_t SELECT_CLASS[] = L"GameSelectWindowClass";
            WNDCLASS wc = {};
            wc.lpfnWndProc = GameSelectProc;
            wc.hInstance = (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE);
            wc.lpszClassName = SELECT_CLASS;
            RegisterClass(&wc);

            hGameSelectWnd = CreateWindowEx(
                0,
                SELECT_CLASS,
                L"게임 선택",
                WS_OVERLAPPEDWINDOW,
                CW_USEDEFAULT, CW_USEDEFAULT, 400, 400,
                hwnd,
                NULL,
                (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE),
                NULL
            );

            ShowWindow(hGameSelectWnd, SW_SHOW);
            break;
        }
        case BUTTON_SETTINGS: {
            const wchar_t SETTINGS_CLASS[] = L"SettingsWindowClass";
            WNDCLASS wc = {};
            wc.lpfnWndProc = SettingsProc;
            wc.hInstance = (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE);
            wc.lpszClassName = SETTINGS_CLASS;
            RegisterClass(&wc);

            hSettingsWnd = CreateWindowEx(
                0,
                SETTINGS_CLASS,
                L"설정",
                WS_OVERLAPPEDWINDOW,
                CW_USEDEFAULT, CW_USEDEFAULT, 400, 300,
                hwnd,
                NULL,
                (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE),
                NULL
            );

            ShowWindow(hSettingsWnd, SW_SHOW);
            break;
        }
        case BUTTON_EXIT:
            PostQuitMessage(0);
            break;
        }
        break;
    }
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

// 설정 화면의 메시지 처리
LRESULT CALLBACK SettingsProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 전체 화면 체크 박스
        CreateWindow(
            L"BUTTON", L"전체 화면",
            WS_VISIBLE | WS_CHILD | BS_CHECKBOX,
            50, 50, 120, 20,
            hwnd, (HMENU)CHECKBOX_FULLSCREEN, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);

        // 대화 속도 슬라이더
        CreateWindow(
            L"STATIC", L"대화 속도:",
            WS_VISIBLE | WS_CHILD,
            50, 100, 80, 20,
            hwnd, NULL, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        HWND hDialogueSlider = CreateWindow(
            TRACKBAR_CLASS, NULL,
            WS_CHILD | WS_VISIBLE | TBS_AUTOTICKS,
            130, 100, 200, 30,
            hwnd, (HMENU)SLIDER_DIALOGUE_SPEED, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        SendMessage(hDialogueSlider, TBM_SETRANGE, TRUE, MAKELPARAM(1, 10));

        // 캐릭터 이동 속도 슬라이더
        CreateWindow(
            L"STATIC", L"이동 속도:",
            WS_VISIBLE | WS_CHILD,
            50, 150, 80, 20,
            hwnd, NULL, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        HWND hMovementSlider = CreateWindow(
            TRACKBAR_CLASS, NULL,
            WS_CHILD | WS_VISIBLE | TBS_AUTOTICKS,
            130, 150, 200, 30,
            hwnd, (HMENU)SLIDER_MOVEMENT_SPEED, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        SendMessage(hMovementSlider, TBM_SETRANGE, TRUE, MAKELPARAM(1, 10));

        // 사운드 조절 슬라이더
        CreateWindow(
            L"STATIC", L"사운드:",
            WS_VISIBLE | WS_CHILD,
            50, 200, 80, 20,
            hwnd, NULL, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        HWND hSoundSlider = CreateWindow(
            TRACKBAR_CLASS, NULL,
            WS_CHILD | WS_VISIBLE | TBS_AUTOTICKS,
            130, 200, 200, 30,
            hwnd, (HMENU)SLIDER_SOUND_VOLUME, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        SendMessage(hSoundSlider, TBM_SETRANGE, TRUE, MAKELPARAM(0, 100));

        // 뒤로 가기 버튼
        CreateWindow(
            L"BUTTON", L"뒤로",
            WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            150, 250, 100, 30,
            hwnd, (HMENU)BUTTON_BACK, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        break;
    }
    case WM_COMMAND: {
        if (LOWORD(wParam) == BUTTON_BACK) {
            ShowWindow(hwnd, SW_HIDE);
        }
        break;
    }
    case WM_DESTROY:
        hSettingsWnd = NULL;
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

// 게임 선택 화면의 메시지 처리
LRESULT CALLBACK GameSelectProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 8개의 게임 선택 버튼 생성
        const wchar_t* buttonTitles[8] = {
            L"조융전", L"준비 중", L"준비 중", L"준비 중",
            L"준비 중", L"준비 중", L"준비 중", L"준비 중"
        };

        for (int i = 0; i < 8; ++i) {
            CreateWindow(
                L"BUTTON", buttonTitles[i],
                WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
                50 + (i % 2) * 150, 50 + (i / 2) * 70, 120, 40,
                hwnd, (HMENU)(BUTTON_GAME_START + i), (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        }
        break;
    }
    case WM_COMMAND: {
        int wmId = LOWORD(wParam);
        if (wmId == BUTTON_GAME_START) {  // 조융전 버튼 클릭 시
            // 스테이지 선택 창 열기
            const wchar_t STAGE_CLASS[] = L"StageSelectWindowClass";
            WNDCLASS wc = {};
            wc.lpfnWndProc = StageSelectProc; // 스테이지 선택 프로시저 지정
            wc.hInstance = (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE);
            wc.lpszClassName = STAGE_CLASS;
            RegisterClass(&wc);

            hStageSelectWnd = CreateWindowEx(
                0,
                STAGE_CLASS,
                L"스테이지 선택",
                WS_OVERLAPPEDWINDOW,
                CW_USEDEFAULT, CW_USEDEFAULT, 400, 300,
                hwnd,
                NULL,
                (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE),
                NULL
            );

            ShowWindow(hStageSelectWnd, SW_SHOW);
        }
        else {
            MessageBox(hwnd, L"이 기능은 준비 중입니다.", L"알림", MB_OK);
        }
        break;

    }
    case WM_DESTROY:
        hGameSelectWnd = NULL;
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

// 스테이지 선택 화면의 메시지 처리
LRESULT CALLBACK StageSelectProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 대흥산 전투 버튼
        CreateWindow(
            L"BUTTON", L"대흥산 전투",
            WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
            50, 50, 150, 40,
            hwnd, (HMENU)(BUTTON_STAGE_1), (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);

        // 개발 중 버튼들
        for (int i = 2; i < 4; ++i) {
            CreateWindow(
                L"BUTTON", L"개발 중",
                WS_TABSTOP | WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
                50, 100 + ((i - 1) * 50), 150, 40,
                hwnd, (HMENU)(BUTTON_STAGE_2 + i), (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);
        }
        break;
    }
    case WM_COMMAND: {
        int wmId = LOWORD(wParam);
        if (wmId == BUTTON_STAGE_1) {  // 대흥산 전투 버튼 클릭 시
            // 스토리 화면 열기
            const wchar_t STORY_CLASS[] = L"StoryWindowClass";
            WNDCLASS wc = {};
            wc.lpfnWndProc = StoryProc;
            wc.hInstance = (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE);
            wc.lpszClassName = STORY_CLASS;
            RegisterClass(&wc);

            hStoryWnd = CreateWindowEx(
                0,
                STORY_CLASS,
                L"대흥산 전투 스토리",
                WS_OVERLAPPEDWINDOW,
                CW_USEDEFAULT, CW_USEDEFAULT, 400, 300,
                hwnd,
                NULL,
                (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE),
                NULL
            );

            ShowWindow(hStoryWnd, SW_SHOW);
        }
        else {
            MessageBox(hwnd, L"이 기능은 준비 중입니다.", L"알림", MB_OK);
        }
        break;
    }
    case WM_DESTROY:
        hStageSelectWnd = NULL;
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

// 스토리 내용 배열
const wchar_t* stories[] = {
    L"추정:조융 장수님! 당신이 오셨군요. 황건적의 위협이 심각해져가고 있습니다. 저와 함께 힘을 모아야 합니다.",
    L"조융:물론입니다, 추정. 우리가 힘을 합쳐야 합니다.",
    L"추정:의용병분들! 조융 장수님을 소개합니다. 이분은 우리를 도와주러 오신 분입니다.",
    L"유비:조융 장수님, 처음 뵙겠습니다. 저는 유비입니다. 함께 싸울 수 있어 기쁩니다.",
    L"추정:순서대로 유비, 관우, 장비, 장경입니다.",
    L"조융:그렇군요. 이런 시대의 의용병이라니 대단하군요.",
    L"조융과 유비는 대흥산에서 연을 틀었다.",
    L"잠시 후...",
    L"장경:적들이 처들어 왔습니다.",
    L"추정:이런, 큰일이구만",
    L"유비:저희가 나가보겠습니다.",
    L"조융:저도 나가겠습니다.",
    L"스토리 끝입니다..."
};

int currentStoryIndex = 0; // 현재 스토리 인덱스
HWND hEdit; // 텍스트 상자 핸들

#include <windows.h>
#include <vector>
#include <string>

// UnitType 열거형 정의
enum class UnitType {
    SpecialInfantry, // 특수보병계
    Lord,            // 군주계
    Cavalry,         // 기병계
    Infantry,        // 보병계
    Rebel,           // 황건적계
    Scholar            // 책사계
};

// Character 구조체 정의
struct Character {
    std::wstring name;    // 캐릭터 이름 (유니코드 지원)
    int health;           // 캐릭터의 체력
    bool isControllable;  // 조종 가능 여부
    int level;            // 캐릭터의 레벨
    int experience;       // 캐릭터의 경험치
    UnitType unitType;    // 병과 타입
    int attackPower;      // 공격력
    int mentalPower;      // 정신력
    int defensePower;     // 방어력
    int agility;          // 순발력
    int morale;           // 사기

    // 생성자
    Character(std::wstring n, int h, bool control, int lvl, UnitType type, int attack, int mental, int defense, int agilityVal, int moraleVal)
        : name(n), health(h), isControllable(control), level(lvl), experience(0), unitType(type),
        attackPower(attack), mentalPower(mental), defensePower(defense), agility(agilityVal), morale(moraleVal) {}

    // 레벨업 함수
    void LevelUp() {
        level++;
        // 병과에 따라 스탯 증가량 다르게 설정
        switch (unitType) {
        case UnitType::SpecialInfantry: // 특수보병
            health += 12;
            attackPower += 3;
            mentalPower += 3;
            defensePower += 3;
            agility += 3;
            morale += 3;
            break;
        case UnitType::Lord: // 군주
            health += 12;
            attackPower += 3;
            mentalPower += 3;
            defensePower += 3;
            agility += 3;
            morale += 3;
            break;
        case UnitType::Scholar: // 책사
            health += 8;
            attackPower += 2;
            mentalPower += 3;
            defensePower += 2;
            agility += 2;
            morale += 2;
            break;
        case UnitType::Cavalry: // 기병
            health += 12;
            attackPower += 3;
            mentalPower += 2;
            defensePower += 3;
            agility += 2;
            morale += 2;
            break;
        case UnitType::Infantry: // 보병
            health += 12;
            attackPower += 2;
            mentalPower += 3;
            defensePower += 3;
            agility += 2;
            morale += 2;
            break;
        case UnitType::Rebel: // 황건적
            health += 10;
            attackPower += 2;
            mentalPower += 1;
            defensePower += 2;
            agility += 2;
            morale += 1;
            break;
        default:
            break;
        }

        MessageBox(NULL, (name + L" 레벨업! 현재 레벨: " + std::to_wstring(level) +
            L" | 체력: " + std::to_wstring(health) +
            L" | 공격력: " + std::to_wstring(attackPower) +
            L" | 정신력: " + std::to_wstring(mentalPower) +
            L" | 방어력: " + std::to_wstring(defensePower) +
            L" | 순발력: " + std::to_wstring(agility) +
            L" | 사기: " + std::to_wstring(morale)).c_str(),
            L"레벨업!", MB_OK);
    }

    // 경험치 추가 함수
    void AddExperience(int xp) {
        experience += xp;
        // 레벨업 조건 확인 (예: 100 경험치마다 레벨업)
        if (experience >= 100 * level) {
            experience -= 100 * level; // 남은 경험치 계산
            LevelUp(); // 레벨업 호출
        }
    }
};

#include <windows.h>

// 색상 정의
#define COLOR_GREEN RGB(0, 255, 0) // RGB(레드, 그린, 블루)
#define COLOR_GRAY RGB(128, 128, 128) // 회색

// 예시로 사용할 색상 상수를 구조체에 포함할 수 있습니다.
struct MyColors {
    COLORREF green;
    COLORREF gray;

    MyColors() : green(COLOR_GREEN), gray(COLOR_GRAY) {}
};

// 사용 예제
void SomeFunction() {
    MyColors colors;

    // 사용 예
    HDC hdc = GetDC(NULL); // 전체 화면의 DC를 가져옴
    SetPixel(hdc, 50, 50, colors.green); // (50, 50) 위치에 녹색 점을 찍음
    SetPixel(hdc, 100, 100, colors.gray); // (100, 100) 위치에 회색 점을 찍음
    ReleaseDC(NULL, hdc); // DC 해제
}


// 맵 타일 종류 정의
enum class TileType {
    Empty,
    Grass,
    Water,
    Wall
};

// 타일 클래스 정의
class Tile {
public:
    TileType type;

    Tile(TileType t) : type(t) {}
};

// 맵 클래스 정의
class Map {
public:
    const int width;
    const int height;
    std::vector<std::vector<Tile>> tiles;

    Map(int w, int h) : width(w), height(h), tiles(h, std::vector<Tile>(w, Tile(TileType::Empty))) {}

    void Initialize() {
        for (int y = 0; y < height; ++y) {
            for (int x = 0; x < width; ++x) {
                // 예시: 홀수 좌표에 풀, 짝수 좌표에 벽 타일 배치
                if ((x + y) % 2 == 0) {
                    tiles[y][x] = Tile(TileType::Grass);
                }
                else {
                    tiles[y][x] = Tile(TileType::Wall);
                }
            }
        }
    }

    void Render(HDC hdc) {
        // 타일 렌더링 로직 (타일 타입에 따라 적절한 색상으로 그리기)
        for (int y = 0; y < height; ++y) {
            for (int x = 0; x < width; ++x) {
                RECT rect;
                rect.left = x * 40;  // 각 타일의 너비
                rect.top = y * 40;   // 각 타일의 높이
                rect.right = rect.left + 40;
                rect.bottom = rect.top + 40;

                if (tiles[y][x].type == TileType::Grass) {
                    FillRect(hdc, &rect, (HBRUSH)(COLOR_GREEN + 1)); // 풀 타일
                }
                else if (tiles[y][x].type == TileType::Wall) {
                    FillRect(hdc, &rect, (HBRUSH)(COLOR_GRAY + 1)); // 벽 타일
                }
            }
        }
    }
};

// 전투 캐릭터 리스트
std::vector<Character> battleCharacters = {
    Character(L"조융", 144, true, 3, UnitType::SpecialInfantry, 16, 16, 16, 16, 16),
    Character(L"유비", 198, false, 5, UnitType::Lord, 10, 9, 4, 7, 7),
    Character(L"관우", 188, false, 5, UnitType::Cavalry, 14, 8, 5, 9, 5),
    Character(L"장비", 188, false, 5, UnitType::Cavalry, 13, 7, 4, 8, 4),
    Character(L"장경", 188, false, 5, UnitType::Cavalry, 11, 6, 5, 9, 3),
    Character(L"등무", 80, false, 1, UnitType::Rebel, 10, 5, 4, 6, 3),
    Character(L"정원지", 80, false, 1, UnitType::Rebel, 10, 5, 3, 5, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
    Character(L"황건적", 80, false, 1, UnitType::Rebel, 9, 4, 3, 4, 2),
};

// 병과 이름을 반환하는 함수
std::wstring GetUnitTypeName(UnitType type) {
    switch (type) {
    case UnitType::SpecialInfantry: return L"특수 보병";
    case UnitType::Lord: return L"군주";
    case UnitType::Cavalry: return L"기병";
    case UnitType::Infantry: return L"보병";
    case UnitType::Rebel: return L"황건적";
    default: return L"알 수 없는 병과";
    }
}

// 전투 화면 프로시저 선언
LRESULT CALLBACK BattleProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
void RegisterBattleWindowClass(HINSTANCE hInstance);
void StartBattle(HWND hwnd);

// 전투 화면 클래스 등록 함수 정의
void RegisterBattleWindowClass(HINSTANCE hInstance) {
    WNDCLASS wc = { };
    wc.lpfnWndProc = BattleProc; // 전투 프로시저
    wc.hInstance = hInstance; // 현재 인스턴스 핸들
    wc.lpszClassName = L"BattleWindowClass"; // 클래스 이름
    wc.hCursor = LoadCursor(NULL, IDC_ARROW); // 기본 커서
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1); // 배경 색상

    if (!RegisterClass(&wc)) {
        MessageBox(NULL, L"전투 창 클래스를 등록할 수 없습니다!", L"오류", MB_OK | MB_ICONERROR);
    }
}

// 전투 화면 프로시저 정의
LRESULT CALLBACK BattleProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 캐릭터 정보 출력
        std::wstring characterInfo;
        for (const auto& character : battleCharacters) {
            characterInfo += character.name + L" | 레벨: " + std::to_wstring(character.level) +
                L" | 체력: " + std::to_wstring(character.health) +
                L" | 공격력: " + std::to_wstring(character.attackPower) +
                L" | 정신력: " + std::to_wstring(character.mentalPower) +
                L" | 방어력: " + std::to_wstring(character.defensePower) +
                L" | 순발력: " + std::to_wstring(character.agility) +
                L" | 사기: " + std::to_wstring(character.morale) +
                L" | 병과: " + GetUnitTypeName(character.unitType) + L"\n";
        }

        // 승리 조건, 패배 조건 및 제한 턴 수 출력
        std::wstring battleConditions;
        battleConditions += L"[승리 조건]: 적 전멸\n";
        battleConditions += L"[패배 조건]: 조융의 퇴각\n";
        battleConditions += L"[제한 턴 수]: 20 턴\n";

        // 조건 정보 메시지 박스
        MessageBox(hwnd, battleConditions.c_str(), L"전투 조건 정보", MB_OK);

        break;
    }
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}


// 전투 시작 함수
void StartBattle(HWND hwnd) {
    RegisterBattleWindowClass((HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE)); // 인스턴스 등록
    // 전투 화면 윈도우 생성
    HWND hBattleWnd = CreateWindowEx(
        0,
        L"BattleWindowClass", L"전투 화면",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 600, 400,
        hwnd, NULL, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL
    );

    // 현재 스토리 화면을 숨김
    ShowWindow(hwnd, SW_HIDE);
    ShowWindow(hBattleWnd, SW_SHOW); // 전투 화면 보이기
}

// 스토리 화면의 메시지 처리
LRESULT CALLBACK StoryProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        // 스토리를 표시할 텍스트 상자 생성
        hEdit = CreateWindowEx(
            WS_EX_CLIENTEDGE,
            L"EDIT", NULL,
            WS_CHILD | WS_VISIBLE | ES_MULTILINE | ES_READONLY,
            10, 10, 360, 240,
            hwnd, NULL, (HINSTANCE)GetWindowLongPtr(hwnd, GWLP_HINSTANCE), NULL);

        // 초기 스토리 내용 설정
        SetWindowText(hEdit, stories[currentStoryIndex]);
        break;
    }
    case WM_LBUTTONDOWN: { // 마우스 왼쪽 버튼 클릭 이벤트 처리
        currentStoryIndex++;
        if (currentStoryIndex < sizeof(stories) / sizeof(stories[0])) {
            // 다음 스토리 설정
            SetWindowText(hEdit, stories[currentStoryIndex]);
        }
        else {
            // 스토리가 끝났을 때 전투로 이동
            StartBattle(hwnd);  // 전투 시작 함수 호출
            currentStoryIndex = sizeof(stories) / sizeof(stories[0]) - 1; // 마지막 스토리로 설정
        }
        break;
    }
    case WM_DESTROY:
        hStoryWnd = NULL;
        return 0;
    }
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
