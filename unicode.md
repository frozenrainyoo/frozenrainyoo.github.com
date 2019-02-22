### Unicode
 The Unicode Standard is a character coding system designed to support the worldwide interchange, processing, and display of the written texts of the diverse languages and technical disciplines of the modern world.

### Develop for Quartz

테스트 결과 Glyph를 얻어와 CTFontDrawGlyphs함수로 Color Emoji를 그릴 경우 32bit 이상 조합되는 Emoji의 경우 제대로 그려지지 않는 문제가 있었다.

예로 👨‍👩‍👧‍👧 를 그릴 때 Glyph를 얻어와 그리게 되면 👨👩👧👧로 그려진다. 조합이 제대로 안 되고 있다.

아래 Example Code의 CTLine을 통하여 그릴 경우에는 제대로 그려진다.

```
NSGraphicsContext* graphicsContext = [NSGraphicsContext currentContext];
CGContextRef context = graphicsContext.CGContext;

CGSize size = dirtyRect.size;
CGContextTranslateCTM(context, 0, size.height);
CGContextScaleCTM(context, 1, -1);

CTFontRef fontRef = CTFontCreateWithName((CFStringRef)@"AppleColorEmoji", 30.0f, NULL);
CFStringRef keys[] = { kCTFontAttributeName };
CFTypeRef values[] = { fontRef };
// use CoreFoundation
CFDictionaryRef attr = CFDictionaryCreate(NULL, (const void **)&keys, (const void **)&values,
                                            sizeof(keys) / sizeof(keys[0]), &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
CFAttributedStringRef attString = CFAttributedStringCreate(NULL, CFSTR("👨‍👩‍👧‍👧🇰🇷💪🏾👩🏿‍🦳"), attr);
CFRelease(attr);

// use NS
// NSDictionary *attrDictionary = [NSDictionary dictionaryWithObjectsAndKeys:(id)fontRef, (NSString *)kCTFontAttributeName, nil];
// NSAttributedString *attString = [[NSAttributedString alloc] initWithString:@"👨‍👩‍👧‍👧🇰🇷💪🏾👩🏿‍🦳" attributes:attrDictionary];

CTLineRef line = CTLineCreateWithAttributedString((CFAttributedStringRef)attString);

CGAffineTransform m = CGAffineTransformMake(1, 0, 0, -1, 0, 0);
CGContextSetTextMatrix(context, m);
CGContextSetTextPosition(context, 50.0, 100.0);
CTLineDraw(line, context);

CFRelease(line);
CFRelease(attString);
// use NS
// [attString release];
CFRelease(fontRef);
```

아래 코드는 CTFontDrawGlyphs를 사용한 코드이다.

```
CFStringRef text = CFSTR("👩🏿‍🦳");
int length = (int)CFStringGetLength(text);
UniChar* unichar = new UniChar[length];
CFRange range = {0, length};
CFStringGetCharacters(text, range, unichar);

CTFontRef ctFont = CTFontCreateWithName(CFSTR("AppleColorEmoji"), 36, NULL);
CGGlyph* glyphs = new CGGlyph[length];
CTFontGetGlyphsForCharacters(ctFont, unichar, glyphs, length);
CGPoint* positions = new CGPoint[length];
CGSize* advs = new CGSize[length];
CTFontGetAdvancesForGlyphs(ctFont, kCTFontOrientationHorizontal, glyphs, advs, length);

CGFloat width = 0;
int index = 0;
while (index < length) {
    positions[index] = CGPointMake(width, 0.0);
    width += advs[index].width;
    ++index;
}
CGAffineTransform m = CGAffineTransformMake(1, 0, 0, -1, 0, 0);
CGContextSetTextMatrix(context, m);
CGContextSetTextPosition(context, 100.0, 100.0);
CTFontDrawGlyphs(ctFont, glyphs, positions, length, context);
CFRelease(ctFont);
```

### Color Emoji
하나 이상의 Glyph로 특정한 합성 Glyph를 만들 수 있습니다.  
예를 들어 국기, 사람 피부색 등을 변경할 수 있습니다.

👨👩👧👧 조합으로 👨‍👩‍👧‍👧를 만들 수 있습니다.  

#### History

애모지는 일본에서 시작되었다. 구글이 애모지 도입하면서 유니코드화 프로젝트가 시동되었다.  
2014년 애플과 구글에서 인종의 다양성 존중을 위해 애모지의 피부색을 선택할 수 있는 유니코드 표준에 추가할 것을 제안하고 유니코드 8.0부터 피부색 선택자를 추가하여 조합할 수 있게 되었다.  
(👶(U+1F476) 바로 뒤에 갈색 피부색 선택자 🏿(U+1F3FF)를 추가하면 흑인 아기👶🏿가 표시되는 식)  
피부색 5가지는 제공한다. (GID 356~360)

키스, 커플 등 2명이 나오는 이모지는 남x남, 남x여, 여x여(👨‍❤️‍💋‍👨,👩‍❤️‍💋‍👨,👩‍❤️‍💋‍👩)로 3가지가 존재하며 가족 이모지는 부모의 수(1~2명)와 성별, 아이의 수(1~2명)와 성별까지 따로 둬 총 25가지가 존재한다(👨‍👩‍👦,👨‍👩‍👧,👨‍👩‍👧‍👦,👨‍👩‍👦‍👦,👨‍👩‍👧‍👧,👨‍👨‍👦,👨‍👨‍👧,👨‍👨‍👧‍👦,👨‍👨‍👦‍👦,👨‍👨‍👧‍👧,👩‍👩‍👦,👩‍👩‍👧,👩‍👩‍👧‍👦,👩‍👩‍👦‍👦,👩‍👩‍👧‍👧,👨‍👦,👨‍👦‍👦,👨‍👧,👨‍👧‍👦,👨‍👧‍👧,👩‍👦,👩‍👦‍👦,👩‍👧,👩‍👧‍👦,👩‍👧‍👧). 이걸 하나하나 다 코드를 준 건 아니고, 조합형처럼 여러 이모지 사이에 조합 문자(U+200D Zero Width Joiner)를 넣어서 표기한다. 그 때문에 위 이모지들도 플랫폼에 따라 👨/👩와 🍳이 따로 입력되어 있는 것처럼 보일 수 있다.

마찬가지로 조합 문자(U+200D Zero Width Joiner)를 이용해 이모지 문자와 피부색 문자(🏻🏼🏽🏾🏿)를 붙여 사용한다. 따라서 위 이모지들 역시 👍와 피부색 문자가 따로 입력된 것처럼 보일 수 있다.

위 내용은 아래 위키사이트의 내용을 발췌하였습니다.  
https://namu.wiki/w/%EC%9D%B4%EB%AA%A8%EC%A7%80

### References

https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/CoreText_Programming/Introduction/Introduction.html#//apple_ref/doc/uid/TP40005533-CH1-SW1

https://en.wikipedia.org/wiki/Core_Text

https://www.invasivecode.com/weblog/core-text
