---
layout: post
title: "Salesforce Regex for Emojis"
categories: [All, Technical]
tags: [Salesforce, regex, validation rule, Salesforce regex, emoji]
fullview: false
excerpt: How to prevent user typing emojis into a Salesforce text field
comments: true
---

For a text field, we can't allow users to enter emojis in the text because that field is later ingested in a [MySQL database doesn't have `utf8mb4` support yet](https://mathiasbynens.be/notes/mysql-utf8mb4). 

As a result, we need to add a validation rule to detect emoji and sends an error message.


I ended up using a whitelist approach based on the [unicode categories](https://www.regular-expressions.info/unicode.html):

```
(\p{L}|\p{Sc}|\p{M}|\p{Z}|\p{N}|\p{P}|[\n\r])+
```

With this, you can compose your own validation rules.

Because [Salesforce REGEX formula requires escaping backslashes and other quirks](https://help.salesforce.com/articleView?id=customize_functions_i_z.htm&type=5), it's tricky to compose the regex for emoji. For example, the validation file looks like:

```xml
<ValidationRule>
    <fullName>My_Field_Validation</fullName>
    <active>true</active>
    <description>My_Field validation rule</description>
    <errorConditionFormula>AND(
      LEN (My_Field__c) &gt; 0,
      NOT(REGEX(My_Field__c, &quot;(\\p{L}|\\p{M}|\\p{Sc}|\\p{Z}|\\p{N}|\\p{P}|[\\n\\r])+&quot;))
    )
    </errorConditionFormula>
    <errorDisplayField>My_Field__c</errorDisplayField>
    <errorMessage>Special characters are not supported</errorMessage>
</ValidationRule>
```

**Update**:

I found the emoji source from [here](https://www.regextester.com/106421). To translate it into Salesforce / Java 6 Syntax, the regex looks like below

```
(\u00a9|\u00ae|[\u2000-\u3300]|[\ud83c\ud000-\ud83c\udfff]|[\ud83d\ud000-\ud83d\udfff]|[\ud83e\ud000-\ud83e\udfff])
```

So the final validatation rule looks like:

```xml
<ValidationRule>
    <fullName>My_Field_Validation</fullName>
    <active>true</active>
    <description>My_Field validation rule</description>
    <errorConditionFormula>AND(
      LEN (My_Field__c) &gt; 0,
      REGEX(My_Field__c, &quot;[\\s\\S]*(\\u00a9|\\u00ae|[\\u2000-\\u3300]|[\\ud83c\\ud000-\\ud83c\\udfff]|[\\ud83d\\ud000-\\ud83d\\udfff]|[\\ud83e\\ud000-\\ud83e\\udfff])[\\s\\S]*&quot;)
    )
    </errorConditionFormula>
    <errorDisplayField>My_Field__c</errorDisplayField>
    <errorMessage>Special characters are not supported</errorMessage>
</ValidationRule>
```


Explanations:

- `[\s\S]` is used to match any character, including line limiter. We don't use dot `.` here because `My_Field__c` can be multi-line. See the StackOverflow answer [here](https://stackoverflow.com/a/25071906/1241108).

- `[\ud83c\ud000-\ud83c\udfff]`: We have to add the prefix to both range start and end. That's slightly different than Javascript syntax.


Finally, I looked at the [emoji-regex](https://github.com/mathiasbynens/emoji-regex/blob/master/src/index.js) repo, which uses the Javascript regex: `/<% emojiSequence %>|\p{Emoji_Presentation}|\p{Emoji}\uFE0F|\p{Emoji_Modifier_Base}/`. 
 
 However, Salesforce uses [Java 6 syntax](https://help.salesforce.com/articleView?id=customize_functions_i_z.htm&type=5) which doesn't support character groups like `\p{Emoji}`, so we can't use that regex here.