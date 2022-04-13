---
date: 2021-12-07 20:40:40
layout: post
title: 14. 점진적인 개선
subtitle: 14. 점진적인 개선
description: 14. 점진적인 개선
image: https://leejaedoo.github.io/assets/img/cleancode.png
optimized_image: https://leejaedoo.github.io/assets/img/cleancode.png
category: cleancode
tags:
- cleancode
paginate: true
comments: true
---

확장성이 부족했던 모듈을 개선하고 정리해가는 단계를 보여준다...

여기서는 명령행 인수의 구문을 분석하기 위해 사용하는 Args 유틸리티를 예시로 잘 짜여진 구조에 대해 알아보았다.

* Args의 간단한 사용법

```java
public static void main(String[] args) {
    try {
        Args arg = new Args("l,p#,d*", args);
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d');
        executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.printf("Argument error: %s\n", e.errorMessage());
    }    
}
```

매변수 두 개로 Args 클래스의 인스턴스를 만든다. 첫째 매개변수는 형식 또는 스키마를 지정하는 "l,p#,d*"로 명령행 인수 세 개를 정의한다.<br>
첫 번째 -l은 부울 인수다. 두 번째 -p는 정수 인수다. 세 번째 -d는 문자열 인수다. 두 번째 매개변수는 main으로 넘어온 명령행 인수 배열이다.

인수 값은 getBoolean, getInteger, getString 등과 같은 메서드를 사용하여 가져올 수 있다.

구체적인 오류 메세지는 exception이 제공하는 errorMessage 메서드를 사용한다.

## (잘 짜여진)Args 구현

```java
public class Args {
    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;
    
    public Args(String schema, String[] args) throws ArgsException {
        marshalers = new HashMap<Character, ArgumentMarshaler>();
        argsFound = new HashSet<Charactoer>();
        
        parseSchema(schema);
        parseArgumentStrings(Arrays.asList(args));
    }
    
    private void parseSchema(String schema) throws ArgsException {
        for (String element : schema.split(","))
            if (element.length() > 0)
                parseSchemaElement(element.trim());
    }
    
    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if (elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else if (elementTail.equals("[*]"))
            marshalers.put(elementId, new StringArrayArgumentMarshaler());
        else
            throw new ArguException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
    }

    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId))
            throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null);
    }
    
    private void parseArgumentStrings(List<String> argsList) throws ArgsException {
        for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
            String argString = currentArgument.next();
            if (argString.startsWith("-")) {
                parseArgumentCharacters(argString.substring(1));
            } else {
                currentArgument.previous();
                break;
            }
        }
    }
    private void parseArgumentCharacters(String argChars) throws ArgsException {
        for (int i = 0; i < argChars.lenght(); i++) {
            parseArgumentCharacter(argChars.charAt(i));
        }
    }
    
    private void parseArgumentCharacter(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null) {
            throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
        } else {
            argsFound.add(argChar);
            try {
                m.set(currentArgument);
            } catch (ArgsException e) {
                e.setErrorArgumentId(argChar);
                throw e;
            }
        }
    }

    public boolean has(char args) {
        return argsFound.contains(arg);
    }
    public int nextArgument() {
        return currentArgument.nextIndex();
    }
    public boolean getBoolean(char args) {
        return BooleanArgumentMarshaler.getValue(marshalers.get(args));
    }
    public String getString(char arg) {
        return StringArgumentMarshaler.getValue(marshalers.get(arg));
    }
    public int getInt(char arg) {
        return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
    }
    public double getDouble(char arg) {
        return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
    }
    public String[] getStringArray(char arg) {
        return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
    }
}
```

위에서 부터 아래로 읽힌다는 사실에 주목한다.

아래는 ArgumentMarshaler 인터페이스와 파생 클래스다.

* ArgumentMarshaler

```java
public interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
}
```

* BooleanArgumentMarshaler

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler {
    private boolean booleanValue = false;
    
    @Override
    public void set(Iterator<String> currentArgument) throws ArgsException {
        booleanValue = true;
    }
    
    public static boolean getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler) am).booleanValue;
        else
            return false;
    }
}
```

* StringArgumentMarshaler

```java
public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    @Override
    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_STRING);
        }
    }
    
    public static String getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler) am).stringValue;
        else
            return "";
        
    }
}
```

* IntegerArgumentMarshaler

```java
public class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;

    @Override
    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INVALID_INTEGER, parameter);
        }
    }
    
    public static int getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler) am).intValue;
        else
            return 0;
    }
}
```

* ArgsException(오류 코드 처리)

```java
public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;
    
    public ArgsException() {}
    
    public ArgsException(String message) {
        super(message);
    }
    
    public ArgsException(ErrorCode errorCode, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }
    
    public ArgsException(ErrorCode errorCode, char errorArgumentId, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }
    
    public char getErrorArgumentId() {
        return errorArgumentId;
    }
    
    public void setErrorArgumentId(char errorArgumentId) {
        this.errorArgumentId = errorArgumentId;
    }
    
    public String getErrorParameter() {
        return errorParameter;
    }
    
    public void setErrorParameter(String errorParameter) {
        this.errorParameter = errorParameter;
    }
    
    public ErrorCode getErrorCode() {
        return errorCode;
    }
    
    public void setErrorCode(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }

    public String errorMessage() {
        switch (errorCode) {
            case OK:
                return "TILT: Should not get here.";
            case UNEXPECTED_ARGUMENT:
                return String.format("Argument -%c unexpected.", errorArgumentId);
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.", errorArgumentId);
            case INVALID_INTEGER:
                return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format("Could not find integer parameter for -%c.", errorArgumentId);
            case INVALID_DOUBLE:
                return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
                return String.format("Could not find double parameter for -%c.", errorArgumentId);
            case INVALID_ARGUMENT_NAME:
                return String.format("'%c' is not a valid argument name.", errorArgumentId);
            case INVALID_ARGUMENT_FORMAT:
                return String.format("'%s' is not a valid argument format.", errorParameter);
        }
        return "";
    }

    public enum ErrorCode {
        OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
        MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, MISSING_DOUBLE, INVALID_DOUBLE
    }

}
```

단순한 개념을 이렇게까지 많은 양의 코드를 짜야한다는 것에 놀라울 수 있다. 이렇게 될 수 밖에 없는 이유는 사용 언어가 자바이기 때문이다.

여기서 주목해봐야 할 부분은 `변수명을 지은 방식`, `함수 크기`, `코드 형식`이다.   

만약, 날짜 인수나 복소수 인수 등 새로운 인수 유형을 추가해야 할 일이 생기면 간단하게 추가할 수 있다. 고칠 코드도 별로 없다. 간단히 설명하자면, ArgumentMarshaler에서 새 클래스를 구현하여 getXXX 함수를 추가한 후 parseSchemaElement 함수에 새 case 문만 추가하면 끝이다. 필요하다면 새 ArgsException.ErrorCode를 만들고 새 오류 메시지를 추가한다.

### 어떻게 짜는것인가

위 코드 처럼 한 번에 깔끔한 코드가 나오진 않는다. 프로그래밍은 과학보다는 공예에 가깝다(클린 코드를 짜는데 있어서는 더욱)

> 깨끗한 코드를 짜려면 먼저 1차 초안으로 지저분한 코드를 짠 뒤에 2차로 정리하는 과정을 꼭 가져야 한다.

* 1차 초안

```java
public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>(); 
  private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
  private Map<Character, String> stringArgs = new HashMap<Character, String>(); 
  private Map<Character, Integer> intArgs = new HashMap<Character, Integer>(); 
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;
  
  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}
    
  public Args(String schema, String[] args) throws ParseException { 
    this.schema = schema;
    this.args = args;
    valid = parse();
  }
  
  private boolean parse() throws ParseException { 
    if (schema.length() == 0 && args.length == 0)
      return true; 
    parseSchema(); 
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }
  
  private boolean parseSchema() throws ParseException { 
    for (String element : schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim(); 
        parseSchemaElement(trimmedElement);
      } 
    }
    return true; 
  }
  
  private void parseSchemaElement(String element) throws ParseException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail)) 
      parseBooleanSchemaElement(elementId);
    else if (isStringSchemaElement(elementTail)) 
      parseStringSchemaElement(elementId);
    else if (isIntegerSchemaElement(elementTail)) 
      parseIntegerSchemaElement(elementId);
    else
      throw new ParseException(String.format("Argument: %c has invalid format: %s.", 
        elementId, elementTail), 0);
    } 
  }
    
  private void validateSchemaElementId(char elementId) throws ParseException { 
    if (!Character.isLetter(elementId)) {
      throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
    }
  }
  
  private void parseBooleanSchemaElement(char elementId) { 
    booleanArgs.put(elementId, false);
  }
  
  private void parseIntegerSchemaElement(char elementId) { 
    intArgs.put(elementId, 0);
  }
  
  private void parseStringSchemaElement(char elementId) { 
    stringArgs.put(elementId, "");
  }
  
  private boolean isStringSchemaElement(String elementTail) { 
    return elementTail.equals("*");
  }
  
  private boolean isBooleanSchemaElement(String elementTail) { 
    return elementTail.length() == 0;
  }
  
  private boolean isIntegerSchemaElement(String elementTail) { 
    return elementTail.equals("#");
  }
  
  private boolean parseArguments() throws ArgsException {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
      String arg = args[currentArgument];
      parseArgument(arg); 
    }
    return true; 
  }
  
  private void parseArgument(String arg) throws ArgsException { 
    if (arg.startsWith("-"))
      parseElements(arg); 
  }
  
  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i)); 
  }
  
  private void parseElement(char argChar) throws ArgsException { 
    if (setArgument(argChar))
      argsFound.add(argChar); 
    else 
      unexpectedArguments.add(argChar); 
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT; 
      valid = false;
  }
  
  private boolean setArgument(char argChar) throws ArgsException { 
    if (isBooleanArg(argChar))
      setBooleanArg(argChar, true); 
    else if (isStringArg(argChar))
      setStringArg(argChar); 
    else if (isIntArg(argChar))
      setIntArg(argChar); 
    else
      return false;
    
    return true; 
  }
  
  private boolean isIntArg(char argChar) {
    return intArgs.containsKey(argChar);
  }
  
  private void setIntArg(char argChar) throws ArgsException { 
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      intArgs.put(argChar, new Integer(parameter)); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_INTEGER;
      throw new ArgsException();
    } catch (NumberFormatException e) {
      valid = false;
      errorArgumentId = argChar; 
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER; 
      throw new ArgsException();
    } 
  }
  
  private void setStringArg(char argChar) throws ArgsException { 
    currentArgument++;
    try {
      stringArgs.put(argChar, args[currentArgument]); 
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_STRING; 
      throw new ArgsException();
    } 
  }
  
  private boolean isStringArg(char argChar) { 
    return stringArgs.containsKey(argChar);
  }
  
  private void setBooleanArg(char argChar, boolean value) { 
    booleanArgs.put(argChar, value);
  }
  
  private boolean isBooleanArg(char argChar) { 
    return booleanArgs.containsKey(argChar);
  }
  
  public int cardinality() { 
    return argsFound.size();
  }
  
  public String usage() { 
    if (schema.length() > 0)
      return "-[" + schema + "]"; 
    else
      return ""; 
  }
  
  public String errorMessage() throws Exception { 
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case UNEXPECTED_ARGUMENT:
        return unexpectedArgumentMessage();
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
    }
    return ""; 
  }
  
  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -"); 
    for (char c : unexpectedArguments) {
      message.append(c); 
    }
    message.append(" unexpected.");
    
    return message.toString(); 
  }
  
  private boolean falseIfNull(Boolean b) { 
    return b != null && b;
  }
  
  private int zeroIfNull(Integer i) { 
    return i == null ? 0 : i;
  }
  
  private String blankIfNull(String s) { 
    return s == null ? "" : s;
  }
  
  public String getString(char arg) { 
    return blankIfNull(stringArgs.get(arg));
  }
  
  public int getInt(char arg) {
    return zeroIfNull(intArgs.get(arg));
  }
  
  public boolean getBoolean(char arg) { 
    return falseIfNull(booleanArgs.get(arg));
  }
  
  public boolean has(char arg) { 
    return argsFound.contains(arg);
  }
  
  public boolean isValid() { 
    return valid;
  }
  
  private class ArgsException extends Exception {
  } 
}
```

첫 버전이던 Boolean 인수만 지원하던 초기 버전에서 String과 Integer 인수 유형을 추가하면서 부터 재앙이 시작됐다.

추가할 인수 유형이 적어도 두 개는 더 있었는데 그러면 코드가 훨씬 더 나빠지리라는 사실이 자명했다. 계속 밀어붙이면 프로그램은 어떻게든 완성하겠지만 그랬다가는 너무 커서 손대기 어려운 골칫거리가 생겨날 참이었다. 코드 구조를 유지보수하기 좋은 상태로 만들려면 지금이 적기라 판단했다.

그래서 나는 기능을 더 이상 추가하지 않기로 결정하고 리팩터링을 시작했다. String, Integer 유형을 추가한 경험에서 나는 새 인수 유형을 추가하려면 주요 지점 세 곳에도 코드를 추가해야 한다는 사실을 이미 깨달았다.

인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나가 적합하다 판단했다. 그래서 ArgumentMarshaler라는 개념이 탄생했다.

### 점진적으로 개선하다

프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 어떤 프로그램은 그저 그런 '개선'에서 결코 회복하지 못한다. '개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다.

그래서 이 때 테스트 주도 개발(Test-Driven Development, TDD)라는 기법을 사용해 볼 수 있다. TDD는 언제 어느 때라도 시스템이 돌아가야 한다는 원칙을 따른다. 다시 말해, `TDD는 시스템을 망가뜨리는 변경을 허용하지 않는다.` 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다는 말이다.

변경 전후 시스템이 똑같이 돌아간다는 사실을 확인하려면 언제든 실행이 가능한 자동화된 테스트 슈트가 필요하다. 앞서 Args 클래스를 구현하는 동안에 나는 이미 단위 테스트 슈트와 인수 테스트를 만들어 놓았다. 두 테스트 모두 언제든 실행이 가능했고, 시스템이 두 테스트를 모두 통과하면 올바로 동작한다고 봐도 좋았다.

그래서 나는 시스템에 자잘한 변경을 가하기 시작했다. 코드를 변경할 때마다 시스템 구조는 조금씩 ArgumentMarshaler 개념에 가까워졌다. 또한 변경 후에도 시스템은 여전히 잘 돌아갔다. 가장 먼저 나는 기존 코드 끝에 ArgumentMarshaler 클래스의 골격을 추가했다. 우선 boolean 부분 부터 해 보기로 했다.

* 끝에 추가한 ArgumentMarshaler

```java
private class ArgumentMarshaler { 
  private boolean booleanValue = false;

  public void setBoolean(boolean value) { 
    booleanValue = value;
  }
  
  public boolean getBoolean() {return booleanValue;} 
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler { }
private class StringArgumentMarshaler extends ArgumentMarshaler { }
private class IntegerArgumentMarshaler extends ArgumentMarshaler { }
```

그리고 코드를 최소로 건드리는, 가장 단순한 변경을 가했다. 구체적으로는 Boolean 인수를 저장하는 HashMap에서 Boolean 인수 유형을 ArgumentMarshaler 유형으로 바꿨다.

```java
private Map<Character, ArgumentMarshaler> boolean Args = new HashMap<Character, ArgumentMarshaler>();
```

```java
//...

private void parseBooleanSchemaElement(char elementId) {
  booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
  
//...

private void setBooleanArg(char argChar, boolean value) {
  booleanArgs.get(argChar).setBoolean(value);
}
  
//...

public boolean getBoolean(char arg) {
  Args.ArgumentMarshaler am = booleanArgs.get(arg);
  return am != null && am.getBoolean();
}
```

* ArgsTest

```java
public class ArgsTest extends TestCase {
    public void testCreateWithNoSchemaOrArguments() throws Exception {
        Args args = new Args("", new String[0]);
        assertEquals(0, args.cardinality());
    }
    
    public void testWithNoSchemaButWithOneArgument() throws Exception {
        try {
            new Args("", new String[]{"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
    public void testWithNoSchemaButWithMultipleArguments() throws Exception {
        try {
            new Args("", new String[]{"-x", "-y"});
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }

    //...
}
```

## 최종 코드

* Args

```java
public class Args {
  private String schema;
  private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
  private Set<Character> argsFound = new HashSet<Character>(); 
  private Iterator<String> currentArgument;
  private List<String> argsList;
  
  public Args(String schema, String[] args) throws ArgsException { 
    this.schema = schema;
    argsList = Arrays.asList(args);
    parse();
  }
  
  private void parse() throws ArgsException { 
    parseSchema();
    parseArguments();
  }
  
  private boolean parseSchema() throws ArgsException {
    for (String element : schema.split(",")) { 
      if (element.length() > 0) {
        parseSchemaElement(element.trim()); 
      }
    }
    return true; 
  }
  
  private void parseSchemaElement(String element) throws ArgsException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1); 
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*")) 
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##")) 
      marshalers.put(elementId, new DoubleArgumentMarshaler());
    else
      throw new ArgsException(ArgsException.ErrorCode.INVALID_FORMAT, elementId, elementTail);
      
  private void validateSchemaElementId(char elementId) throws ArgsException { 
    if (!Character.isLetter(elementId)) {
      throw new ArgsException(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, elementId, null);
    } 
  }
  
  private void parseArguments() throws ArgsException {
    for (currentArgument = argsList.iterator(); currentArgument.hasNext();) {
      String arg = currentArgument.next();
      parseArgument(arg); 
    }
  }
  
  private void parseArgument(String arg) throws ArgsException { 
    if (arg.startsWith("-"))
      parseElements(arg); 
  }
  
  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i)); 
  }
  
  private void parseElement(char argChar) throws ArgsException { 
    if (setArgument(argChar))
      argsFound.add(argChar); 
    else 
      throw new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, argChar, null);
  } 
  
  private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
      return false; 
    try {
      m.set(currentArgument);
      return true;
    } catch (ArgsException e) {
      e.setErrorArgumentId(argChar);
      throw e; 
    }
  }
  
  public int cardinality() { 
    return argsFound.size();
  }
  
  public String usage() { 
    if (schema.length() > 0)
      return "-[" + schema + "]"; 
    else
      return ""; 
  }
  
  public boolean getBoolean(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    boolean b = false;
    try {
      b = am != null && (Boolean) am.get(); 
    } catch (ClassCastException e) {
      b = false; 
    }
    return b; 
  }
  
  public String getString(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? "" : (String) am.get(); 
    } catch (ClassCastException e) {
      return ""; 
    }
  }
  
  public int getInt(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? 0 : (Integer) am.get(); 
    } catch (Exception e) {
      return 0; 
    }
  }
  
  public double getDouble(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? 0 : (Double) am.get(); 
    }
```

## 결론

그저 돌아가는 코드만으로는 부족하다. 돌아가는 코드가 심하게 망가지는 사례는 흔하다. 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다. 설계와 구조를 개선할 시간이 없다고 변명할지 모르지만, 동의하기 어렵다. 나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다. 나쁜 일정은 다시 짜면 된다. 나쁜 요구사항은 다시 정의하면 된다. 나쁜 팀 역학은 복구하면 된다. 하지만 나쁜 코드는 썩어 문드러진다. 점점 무게가 늘어나 팀의 발목을 잡는다. 속도가 점점 느려지다 못해 기어가는 팀도 많이 봤다. 너무 서두르다가 이후로 영원히 자신들의 운명을 지배할 악성 코드라는 굴레를 짊어진다.

물론 나쁜 코드도 깨끗한 코드로 개선할 수 있다. 하지만 비용이 엄청나게 많이 든다. 코드가 썩어가며 모듈은 서로서로 얽히고설켜 뒤엉키고 숨겨진 의존성이 수도 없이 생긴다. 오래된 의존성을 찾아내 깨려면 상당한 시간과 인내심이 필요하다. 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다. 아침에 엉망으로 만든 코드를 오후에 정리하기는 어렵지 않다. 더욱이 5분 전에 엉망으로 만든 코드는 지금 당장 정리하기 아주 쉽다.

그러므로 코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.
