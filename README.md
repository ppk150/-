# -아두이노 + 진공관 디스플레이(닉시관) 프로젝트 (6Digit, IN12-B Type)
#include <ThreeWire.h>  
#include <RtcDS1302.h>
//---------------------------------nixie-----

int select[] = {9,10,11,12};// 디코더 선택핀 D9~12 D9 = A, D10 = B, D11 = C, D12 = G1->항상 on상태로
//int select[] = {9,0,11,12};
int selectnixie[9][4] = {
  {0,0,0,1},
  {1,0,0,1},
  {0,1,0,1},
  {0,0,0,1},
  {1,0,0,1},
  {0,1,0,1},
  {1,1,0,1},
  {0,0,1,1},
  {1,0,1,1}
}; // 디코더 출력 1~6번째 닉시관





int IC[] = {5,6,7,8}; // 닉시 ic 선택핀 D5~D8 D5 = D, D6 = C, D7 = B, D8 = A

// -----------------------시계 변수 
 int hour10,hour1,minute10,minute1,second10,second11,second1;
// -----------------------날짜 변수
 int year10, year1, month10, month1, day10, day1;

// 0~9 닉시 폰트 
int pattern[10][4]={
{0,0,0,0}, // 0
{0,0,0,1}, // 1
{0,0,1,0}, // 2
{0,0,1,1}, // 3
{0,1,0,0}, // 4
{0,1,0,1}, // 5
{0,1,1,0}, // 6
{0,1,1,1}, // 7
{1,0,0,0}, // 8
{1,0,0,1}  // 9  
};

//------------------------------RTC
//A5 19핀이 DAT
//A3 17핀이 CLK
//A4 18핀이 CE
ThreeWire myWire(19,17,18); // IO, SCLK, CE

//RTC 라이브러리 생성
RtcDS1302<ThreeWire> Rtc(myWire);

void setup () 
{
  // -----------nixie-----

  for(int i =0; i<4; i++){
    pinMode(select[i],OUTPUT);
  } // 멀티플렉서로 수정 필요 현제 digit로 되어있음

    for(int i =0; i<4; i++){
    pinMode(IC[i],OUTPUT);
  }
  // -----충격센서
  pinMode(3, INPUT);
  attachInterrupt(INT1, date, RISING);

 //   pinMode(2, INPUT);
 // attachInterrupt(INT0, timeset, RISING);
  
  //--------------------- RTC----
    Serial.begin(57600);
    //컴파일 시점의 날짜(__DATE__)와 시간(__TIME__)을 시리얼모니터에 표시
    Serial.print("compiled: ");
    Serial.print(__DATE__);
    Serial.println(__TIME__);

    //RTC 모듈 라이브러리 시작
    Rtc.Begin();

    //RTCDateTime 클래스 생성(컴파일된 시간으로 설정)
    RtcDateTime compiled = RtcDateTime(__DATE__, __TIME__);
    printDateTime(compiled);
    Serial.println();
    
    //RTC모듈에 쓰기 금지 모드인지 확인
    if (Rtc.GetIsWriteProtected())
    {
        Serial.println("RTC was write protected, enabling writing now");
        //쓰기 금지 모드이면 해제
        Rtc.SetIsWriteProtected(false);
    }
    
    //RTC 모듈이 동작 안하는 상태일 시
    if (!Rtc.GetIsRunning())
    {
        Serial.println("RTC was not actively running, starting now");
        // RTC모듈 시작
        Rtc.SetIsRunning(true);
    }

    // RTC 모듈의 현재 시간 얻기
    RtcDateTime now = Rtc.GetDateTime();

    //RTC 모듈과 위의 클래스 시간과 비교
    //RTC 시간 파악, 클래스 시간보다 늦으면 
    if (now < compiled) 
    {
        //RTC가 시간을 갱신
        Serial.println("RTC is older than compile time!  (Updating DateTime)");
        //컴파일 시간을 RTC 모듈에 적용
        Rtc.SetDateTime(compiled);
    }
    //RTC 모듈이 컴파일한 시간보다 빠르다면
    else if (now > compiled) 
    {
        //RTC가 컴파일 시간보다 최신
        Serial.println("RTC is newer than compile time. (this is expected)");
        Rtc.SetDateTime(compiled);
    }
    //RTC와 컴파일한 시간이 같다
    else if (now == compiled) 
    {
        //RTC와 컴파일 시간이 같을 시... 그래도 괜찮다는 문구 출력
        Serial.println("RTC is the same as compile time! (not expected but all is fine)");
    }
}

//닉시 시간출력함수
void show_time(int digit, int num){
for(int k =0; k<9;k++){
  if(digit ==k+1){
    for(int j = 0;j<4;j++)
    digitalWrite(select[j],selectnixie[k][j]);
  }
  else{
   digitalWrite(select[k],LOW); 
  }
}
for(int i=0; i<10; i++){
  if(num==i){
    for(int j=0; j<4; j++){
      digitalWrite(IC[j],pattern[i][j]);
    }
  }
 }
}

//---- 닉시 시간출력함수 끝

//---- 닉시 날짜출력함수 
 void date (){
  int k =0;
    while( k < 555){
    show_time(4,year10);
    delay(5);
    show_time(5,year1);
    delay(5);
    show_time(6,month10);
    delay(5);
    show_time(7,month1);
    delay(5);
    show_time(8,day10);
    delay(5);
    show_time(9,day1);
    delay(5);
    k++;
    };
 }
//---닉시 날짜 출력함수 끝


// -- 버튼 눌러서 시간 보정 함수 
/*
void timeset(){

 second11 = second10 - 1; 
}
*/
void loop () 
{
    //RTC 모듈의 현재 시간 얻기
    RtcDateTime now = Rtc.GetDateTime();

  
  //----nixie---- 시계 변수 
  
  hour10 = now.Hour()/10;
  hour1 = now.Hour()%10;
  minute10 = now.Minute()/10;
  minute1 = now.Minute()%10;
  second10 = now.Second()/10;
  second1 = now.Second()%10;
  //----------nixie 날짜변수
  year10 = (now.Year()-2000)/10;
  year1 = (now.Year()-2000)%10;
  month10 = now.Month()/10;
  month1 = now.Month()%10;
  day10 = now.Day()/10;
  day1 = now.Day()%10;

         
  //-----------RTC----

    //시계 닉시관 출력 함수 사용 
    show_time(4,hour10);
    delay(1);
    show_time(5,hour1);
    delay(1);
    show_time(6,minute10);
    delay(1);    
    show_time(7,minute1);
    delay(1);
    show_time(8,second10);
    delay(1);
    show_time(9,second1);
    delay(1);
    

    //시리얼 모니터에 출력
    printDateTime(now);
    //줄 바꿈.
    Serial.println();
    //10초 대기...후 다시 loop 시작
    //delay(1000); // ten seconds
}

#define countof(a) (sizeof(a) / sizeof(a[0]))

//시리얼 모니터에 날짜 시간 표시하는 함수
void printDateTime(const RtcDateTime& dt)
{
    char datestring[20];

    snprintf_P(datestring, 
            countof(datestring),
            PSTR("%02u/%02u/%04u %02u:%02u:%02u"),
            dt.Month(),
            dt.Day(),
            dt.Year(),
            dt.Hour(),
            dt.Minute(),
            dt.Second() );
    Serial.print(datestring);
}
