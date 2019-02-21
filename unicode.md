# Color Emoji

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

### References

https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/CoreText_Programming/Introduction/Introduction.html#//apple_ref/doc/uid/TP40005533-CH1-SW1

https://en.wikipedia.org/wiki/Core_Text

https://www.invasivecode.com/weblog/core-text
