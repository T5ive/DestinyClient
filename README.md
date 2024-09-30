
Hi, ragezone  
Im don't have time to develop anymore it's seem like i cannot continue to develop on private maplestory again.  
Today im coming to release my source code to unlock to chat in other language.  
For today i will release for thai language to chat in v176 (server base on swordie)  
  
First thing you need to hook maplestory function on Canvas.dll to unlock langauge to render. you need follow these steps.  
  
***Use injector to inject your dll to hook maplestory client function  
***Do not forget to bypass MSCRC, Remove NGS  
  
Start with Client side  
Part of maplestory.exe (maplestory client)  
  
1.On dllmain you need to force set local for other langauge first  
(eg. your client run on windows with other locale will make use latin character set instead UTF8.  
if i dont set will make 0xe31 in UTF8 to 0xd1 latin character set)  

```
auto locale = setlocale(LC_ALL, NULL);
```

  
2. Hook string pool to replace font first.  

```
typedef void(__fastcall* ZXString_char__Assign_t)(void* pThis, void* edx, const char* s, int n);
ZXString_char__Assign_t ZXString_char__Assign = reinterpret_cast<ZXString_char__Assign_t>(0x0047EDD0); //176
//ZXString_char__Assign_t ZXString_char__Assign = reinterpret_cast<ZXString_char__Assign_t>(0x00823160); //203.4

bool Hook_StringPool__GetString() {

    typedef ZXString<char>* (__fastcall* StringPool__GetString_t)(void* ecx, void* edx, ZXString<char>* result, unsigned int nIdx, char formal);
    static auto StringPool__GetString = reinterpret_cast<StringPool__GetString_t>(0x00EC3BF0); //176
    //static auto StringPool__GetString = reinterpret_cast<StringPool__GetString_t>(0x00C4C980); //203.4

    StringPool__GetString_t Hook = [](void* ecx, void* edx, ZXString<char>* result, unsigned int nIdx, char formal) -> ZXString<char>*
    {
        StringPool__GetString(ecx, edx, result, nIdx, formal);

        if (0 == strcmp(result->m_pStr, "Arial") || 0 == strcmp(result->m_pStr, "Arial Narrow")) //GMS use Arial, Arial Narrow as font to render
        {
            const char* szFontName = "Leelawadee"; //Replace as Leelawadee as font to render
            result->m_pStr = 0;
            ZXString_char__Assign(result, edx, szFontName, -1);
        }

        return result;
    };
    return SetHook(true, reinterpret_cast<void**>(&StringPool__GetString), Hook);
}
```

  
3. Rip off latin check on GMS client Ex. CCtrlEdit::InsertString, TranslateMessage and much more.  
they always check and replace other language to empty.  
If you want to render more than chat you need to rip off check latin in another class.  

```
bool RipOffLatinCheck() {

#pragmaregionv176
    *(BYTE*)(0x0193EDF8 + 0) = 0xFB; //CWndMan::TranslateMessage Enable Latin 0xFB like THMS
    *(BYTE*)(0x006F4325 + 1) = 0xFB; //CCtrlEdit::InsertString
    *(BYTE*)(0x00EC5787 + 1) = 0xFB; //CCurseProcess::IsValidCharacterName
    *(BYTE*)(0x00EC5A7D + 1) = 0xFB;
    *(BYTE*)(0x018C63FF + 1) = 0xFB;
    *(BYTE*)(0x01AD0C44 + 1) = 0xFB;
    *(BYTE*)(0x01AECA15 + 1) = 0xFB;

    //*(BYTE*)(0x00EC5F8A + 1) = 0xFB; //CCurseProcess::IsValidCharacterName
#pragmaendregion

    //#pragma region v203.4
    ////83 ? 20 0F ? ? ? ? ? 83 ? 7F = CWndMan::TranslateMessage - Enable Latin 0xFB like THMS
    //*(BYTE*)(0x029AA0BF + 0) = 0xFB; //replace 0x7F -> 0xFB

    ///*
    //  AOB : 3C E6 74 ? 3C F8  
    //  Description : Xref to below functions
    //  - CUser::OnChat,
    //  - CCurseProcess::InvalidCharacterName,
    //  - CField::OnGroupMessage
    //  - CField::OnGroupMessage_2
    //  - CUser::OnADBoard
    //*/

    ////*(BYTE*)(0x00C4EC0D + 1) = 0xFB; //CCurseProcess::InvalidCharacterName
    //*(BYTE*)(0x0254C694 + 1) = 0xFB; //CUser::OnADBoard

    ////3C 7A 7E - Other latin checks
    ////*(BYTE*)(0x00770624 + 1) = 0xFB;
    ////*(BYTE*)(0x0079E96A + 1) = 0xFB;
    ////*(BYTE*)(0x0079EB40 + 1) = 0xFB;
    ////*(BYTE*)(0x00C4EC0D + 1) = 0xFB;
    ////*(BYTE*)(0x00C4EF08 + 1) = 0xFB;
    ////*(BYTE*)(0x00C4F237 + 1) = 0xFB;
    ////*(BYTE*)(0x00C51EA6 + 1) = 0xFB;
    ////*(BYTE*)(0x0121B17A + 1) = 0xFB;
    ////*(BYTE*)(0x0121B193 + 1) = 0xFB;
    ////*(BYTE*)(0x0285F5C7 + 1) = 0xFB;
    ////*(BYTE*)(0x02B6B916 + 1) = 0xFB;
    ////*(BYTE*)(0x02B85D63 + 1) = 0xFB;

    ////*(BYTE*)(0x00EC5F8A + 1) = 0xFB; //CCurseProcess::IsValidCharacterName
    //#pragma endregion

    return true;
}
```

  
4. Unlock latin check on creation character screen  

```
bool Hook_IsValidCharacterName()
{
    typedef int(__fastcall* CCurseProcess__IsValidCharacterName_t)(void* a1, void* a2, ZXString<char>* sCharacterName, void* a4);
    static auto CCurseProcess__IsValidCharacterName = reinterpret_cast<CCurseProcess__IsValidCharacterName_t>(0x00EC5EF0); //v176
    //static auto CCurseProcess__IsValidCharacterName = reinterpret_cast<CCurseProcess__IsValidCharacterName_t>(0x00C4EB70); //v203.4

    CCurseProcess__IsValidCharacterName_t Hook = [](void* a1, void* a2, ZXString<char>* sCharacterName, void* a4) -> int
    {
        auto validName = CCurseProcess__IsValidCharacterName(a1, a2, sCharacterName, a4);
        if (!validName) {
            auto strLength = *(sCharacterName->m_pStr - 1);
            auto strIndex = 0;
            do {
                auto chr = sCharacterName->m_pStr[strIndex];

                if (chr < 0x61 || chr > 0xFB) {
                    validName = true;
                }
                else {
                    validName = false;
                    break;
                }

                ++strIndex;
            } while (strIndex < strLength);
        }

        return validName;
    };
    return SetHook(true, reinterpret_cast<void**>(&CCurseProcess__IsValidCharacterName), Hook);
}
```

  
5. Remove check latin character before create packet to server!  

```
bool DisablePacketStringCheck() {

#pragmaregionv176

    PatchNop(0x01547C2C, 7); //CUIStatusBar::OnKey
    PatchNop(0x016FD78D, 9); //CUser::OnChat

#pragmaendregion
    //v203 = 74 0C 66 C7 06 20 20 B8 02 00 00 00 EB 1F 8A 06
    /*PatchNop(0x0135A5AF, 7);
    PatchNop(0x0135AB08, 7);
    PatchNop(0x0254B808, 7);
    PatchNop(0x0254C680, 7);*/

    return true;
}
```

  
  
Part of canvas.dll (render)  
  
1. Prepare variable and load address like this  
  
```
DWORD hOrgCanvasBase = (DWORD)LoadLibraryA("Canvas.dll"); //Load canvas.dll base address
```

  
2. Hook CFont::GetLogfont for replace character set. You can looking lfCharSet on microsoft character on below link.  
Ref : [https://docs.microsoft.com/en-us/windows/win32/api/wingdi/ns-wingdi-logfonta](https://docs.microsoft.com/en-us/windows/win32/api/wingdi/ns-wingdi-logfonta)  

```
bool hookCFontGetLogFont() {

    typedef int(__fastcall* CFont__GetLogfont_t)(void* ecx, void* edx, tagLOGFONTA* lf);
    //A8 20
    static auto CFont__GetLogFont = reinterpret_cast<CFont__GetLogfont_t>(hOrgCanvasBase + 0xC8A0); //176
    //static auto CFont__GetLogFont = reinterpret_cast<CFont__GetLogfont_t>(hOrgCanvasBase + 0xD960); //203.4

    CFont__GetLogfont_t Hook = [](void* ecx, void* edx, tagLOGFONTA* lf) -> int {
        //Log("[MapleLog] [CFont::GetLogFont]");
        auto ret = CFont__GetLogFont(ecx, edx, lf);

        lf->lfCharSet = THAI_CHARSET;

        return ret;

    };
    return SetHook(true, reinterpret_cast<void**>(&CFont__GetLogFont), Hook);
}
```

  
*** Above is all for change language on client if your language don't have special character like thai language.  
  

![t2gqdvQ - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums](https://forum.ragezone.com/attachments/t2gqdvq-webp.233892/ "t2gqdvQ - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums")

  
  
3. Prepare more variable for thai langauge  
(thai language have vowels special character for hang above, below character in same position on alphabet example ก็ กุ กิน ไก่.  
if you nore remove width text will be like ก ็ ก ุ ก ิน ไก ่ . that meaning invalid render for thai language.  
that meaning we need to remove some space before render character)  

```
std::vector<int> hangAboveCharacter = { 0xe31, 0xe34, 0xe35, 0xe36, 0xe37, 0xe47, 0xe48, 0xe49, 0xe4a, 0xe4b, 0xe4c, 0xe4d, 0xe4e };
std::vector<int> hangBelowCharacter = { 0xe38, 0xe39, 0xe3a };
std::vector<int> handRightAndTop = { 0xe33 };

std::vector<int> shiftCharacters = { };

DWORD hOrgCanvasBase = (DWORD)LoadLibraryA("Canvas.dll");
const wchar_t* currentDrawText = L"";
int currentDrawTextIndex = 0;
int textLength = 0;
WCHAR currentDrawFontCharacter = 0;

(on init method)
shiftCharacters.insert(shiftCharacters.end(), hangAboveCharacter.begin(), hangAboveCharacter.end());
shiftCharacters.insert(shiftCharacters.end(), hangBelowCharacter.begin(), hangBelowCharacter.end());
shiftCharacters.insert(shiftCharacters.end(), handRightAndTop.begin(), handRightAndTop.end());
```

  
2.Hook CFont::DrawTextA to check latest text maple draw in to client  

```
bool hookCFontDrawTextA() {    
    typedef void(__fastcall* CFont__DrawTextA_t)(void* ecx, void* edx, void* pCanvas, int x, int y, unsigned __int16* sText, int nAlpha, int nTabOrg);    

    //[AOB] 75 12 6A 00 6A 20  
    static auto CFont__DrawTextA = reinterpret_cast<CFont__DrawTextA_t>(hOrgCanvasBase + 0xED20); //v176    
    //static auto CFont__DrawTextA = reinterpret_cast<CFont__DrawTextA_t>(hOrgCanvasBase + 0xC600);  //v203.4    
    
    CFont__DrawTextA_t Hook = [](void* ecx, void* edx, void* pCanvas, int x, int y, unsigned __int16* sText, int nAlpha, int nTabOrg) -> void {               
        if (sText) {           
             //LogW(L"---------------------------------------------------");            
             LogW(L"[MapleLog] [CFont::DrawTextA] x : %u, y: %u, sText : %ls", x, y, sText);            
             //Log("[MapleLog] [CFont::DrawTextA]");            
             currentDrawText = (wchar_t*)sText;            
             textLength = wcslen(currentDrawText);            
             currentDrawTextIndex = 0;        
        }        
        
        CFont__DrawTextA(ecx, edx, pCanvas, x, y, sText, nAlpha, nTabOrg);        
        
        if (sText) {            
            currentDrawText = L""; //Reset            
            textLength = 0;           
            currentDrawTextIndex = 0;        
        }    
    };    
    return SetHook(true, reinterpret_cast<void**>(&CFont__DrawTextA), Hook);
}
```

  
3. Hook CFont::GetFont for what is current character and next character maplestory will draw.  

```
bool hookCFontGetFont() {

    typedef int(__fastcall* CFontGetFont_t)(void* ecx, void* edx, unsigned __int16 c, tagRECT* prc);

    //xref from CFont__Drawfont_t
    static auto CFont__GetFont = reinterpret_cast<CFontGetFont_t>(hOrgCanvasBase + 0xEC00); //v176
    //static auto CFont__GetFont = reinterpret_cast<CFontGetFont_t>(hOrgCanvasBase + 0xD840); //v203.4

    CFontGetFont_t Hook = [](void* ecx, void* edx, unsigned __int16 c, tagRECT* prc) -> int {
        auto ret = CFont__GetFont(ecx, edx, c, prc);

        if (currentDrawText && textLength > 0) { //check for drawble font only :/ (in v203 they get all font in background too)
            int nextPosition = currentDrawTextIndex + 1;
            wchar_t currentCharacter = currentDrawText[currentDrawTextIndex];
            wchar_t nextCharacter = currentDrawText[nextPosition];

            if (nextCharacter && findInArray(nextCharacter, shiftCharacters)) {
                ret = 0; //remove width for special characters
            }

            currentDrawTextIndex = currentDrawTextIndex + 1;
        }

        return ret;

    };
    return SetHook(true, reinterpret_cast<void**>(&CFont__GetFont), Hook);
}
```

  
4. Hook window GDI to remove width on character!!  

```
bool hookTextOutA()
{
    static auto _CreateFontA = decltype(&TextOutA)(GetFunctionAddress("GDI32", "TextOutA"));

    decltype(&TextOutA) Hook = [](HDC hdc, int x, int y, LPCSTR lpString, int c) -> BOOL
    {
        //Log("[MapleLog] [GDI32::TextOutA] ");
        //LogW(L"[MapleLog] [GDI32::TextOutA] lpString : %c, currentDrawFontCharacter : 0x%x, x : %u, y : %u, c : %u", *lpString, currentDrawFontCharacter, x, y, c);

        auto ret = _CreateFontA(hdc, x, y, lpString, c);

        if (hdc && (findInArray(currentDrawFontCharacter, hangAboveCharacter) || findInArray(currentDrawFontCharacter, hangBelowCharacter))) {
            int startX = x + 0;
            int startY = y + 5;

            int endX = startX + 7;
            int endY = startY + 5;

            auto removeColor = RGB(255, 255, 255);

            for (int currentX = startX; currentX < endX; currentX++) {
                for (int currentY = startY; currentY < endY; currentY++) {
                    SetPixel(hdc, currentX, currentY, removeColor);
                }
            }
        }

        return ret;
    };

    return SetHook(true, reinterpret_cast<void**>(&_CreateFontA), Hook);
}
```

  
  
That all for client side. Let go next server side to unlock thai packet.  
1. Go to file : src/main/java/net/swordie/ms/connection/InPacket.java  
and update decodeString method to read string from client  
  
Before  

```
    public String decodeString(int amount) {
        byte[] bytes = decodeArr(amount);
        char[] chars = new char[amount];
        for(int i = 0; i < amount; i++) {
            chars[i] = (char) bytes[i];
        }
        return String.valueOf(chars);
    }
```

After  

```
    public String decodeString(int amount) {
        byte[] bytes = decodeArr(amount);

        return new String(bytes, Charset.forName("MS874")); //Specific your charset
    }
```

  
2. Go to file : src/main/java/net/swordie/ms/connection/OutPacket.java  
and update encodeString to send string to client  
  
Before  

```
public void encodeString(String s, short length) {
        if (s == null) {
            s = "";
        }
        if (s.length() > 0) {
            for (char c : s.toCharArray()) {
                encodeChar(c);
            }
        }
        for (int i = s.length(); i < length; i++) {
            encodeByte((byte) 0);
        }
    }
```

After  

```
public void encodeString(String s, short length) {
        if (s == null) {
            s = "";
        }
        if (s.length() > 0) {
            byte[] aob = s.getBytes(Charset.forName("MS874"));
            encodeArr(aob);
        }
        for (int i = s.length(); i < length; i++) {
            encodeByte((byte) 0);
        }
    }
```

  
3. go to file : src/main/java/net/swordie/ms/util/Util.java  
update to unlock function to validate letter on server.  
  
Before  

```
 public static boolean isDigitLetterString(String str) {
        return str != null && str.matches("[a-zA-Z0-9]+"); // maybe allow special characters?
     }
```

  
After  

```
 public static boolean isDigitLetterString(String str) {
        return str != null && str.matches("[\\u0E00-\\u0E7Fa-zA-Z0-9]+"); // maybe allow special characters?
     }
```

  
  
That completed. run and test your result !!!  
  

![kVxxIU - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums](https://forum.ragezone.com/attachments/kvxxiue-webp.233893/ "kVxxIU - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums")

  
  

![nTO0Ko3 - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums](https://forum.ragezone.com/attachments/nto0ko3-webp.233894/ "nTO0Ko3 - [RELEASE] Allow other language to type (chat and more) [Example Thai langauge] - RaGEZONE Forums")

  
  
  
Full source code in github.  
https://github.com/KururuLABO/DestinyClient  
  
  
Cr. Me  (https://forum.ragezone.com/threads/release-allow-other-language-to-type-chat-and-more-example-thai-langauge.1206111/)
Thanks you YungMoozi in discord to help suggest many thing for this work (i dont know who is he in ragezone hehe so sorry)
