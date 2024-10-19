+++
author = "Damian Kuraś"
title = "Efficient Text Search Algorithms in C++: Knuth-Morris-Pratt" 
date = 2022-07-08
lastmod = 2024-10-19
description = "Learn how the Knuth-Morris-Pratt text search algorithm works and how to implement in C++." 
draft = false
image = "images/kmp-search-thumbnail.png"
keywords = ""
categories = ["Algorithims"]
tags = ["C++","Search"]
slug = "kmp-text-search-algorithm"
math = false
+++

## Introduction

In this post, you'll learn about the Knuth-Morris-Pratt (KMP) algorithm, an efficient text search algorithm faster than a simple naive search.

## Naive String Searching

Let's start with the naive approach to string searching try to match the pattern starting from each character of the text, one by one. When there is a mismatch, we start matching from next letter.

{{< video src="NaiveSearch" controls="false" autoplay="true" loop="true" >}}

While it's straightforward, this method is slow, with a time complexity of **_O_(n\*m)**, where n is the length of the text and m is the length of the pattern.

### Naive String Searching Implementation in C++

```cpp
int NaiveSearch(const std::string &text, const std::string &pattern)
{
    int text_len = text.length();
    int pattern_len = pattern.length();

    if (patternLength == 0) return 0; // Return 0 for an empty pattern
    if (patternLength > textLength) return -1; // Pattern longer than text

    for (int i = 0; i <= text_len - pattern_len ; i++)
    {
        bool isMatch = true;
        for (int j = 0; j < pattern_len; j++)
        {
            if (text[i + j] != pattern[j])
            {
                isMatch = false;
                break;
            }
        }
        if (isMatch)
        {
            return i; // Return the starting index of the match
        }
    }
    return -1; // Return -1 if no match is found
}
```

## Knuth–Morris–Pratt Algorithm (KMP)

The Knuth-Morris-Pratt (KMP) algorithm improves upon the naive string search approach by skipping unnecessary comparisons. The key idea is to use a precomputed table, called the Longest Prefix Suffix (LPS) table, which helps the algorithm efficiently decide how much of the pattern to skip after a mismatch.

### Building the LPS Table

The LPS table is built by iterating over the pattern and calculating, for each position, the length of the longest proper prefix(proper prefix is one that does not encompass the entire string) of the pattern that is also a suffix of the substring ending at that position.

The LPS table helps us know where to "jump" when we find a mismatch during the KMP search. If we have a partial match and a mismatch occurs, the LPS table tells us how much of the previous match we can reuse.

In KMP, suffixes can appear anywhere in the pattern, not just at the end. For example, in the pattern "ABABC", the substring "AB" is a suffix the same as prefix, even though it is not at the very end of the pattern. This allows KMP to skip comparing characters we've already seen.

Here’s how the LPS table is built:

1. We initialize the LPS table with 0 for the first character, since no proper prefix exists for a single character.
2. For each subsequent character in the pattern, we compare it with the previous characters, looking for the longest prefix that is also a suffix.
3. If we find a match, we update the LPS table for that character. If not, we backtrack using the LPS table to find a smaller prefix match.

{{< video src="BuildLPSTable" controls="false" autoplay="true" loop="true" >}}

You can read more about the creation of LPS table [here](https://medium.com/@aakashjsr/preprocessing-algorithm-for-kmp-search-lps-array-algorithm-50e35b5bb3cb)

### How KMP Works: Step-by-Step

Once the LPS table is built, the KMP algorithm uses it to search for the pattern in the text efficiently. Here’s how it works:

1. Start at the beginning of both the text and and the pattern.
2. Compare the current characters of text and the pattern.
3. If they match, continue matching next characters of both of them.
4. If they not match shift current character in pattern based on LPS table.
5. Repeat until the entire text is searched.

{{< video src="KMPSearch" controls="true" autoplay="false" loop="true" >}}

By using the LPS table, the KMP algorithm ensures that the time complexity is **_O_(n + m)**, where n is the length of the text and m is the length of the pattern.

### Implementation in C++.

```cpp
std::vector<int> BuildLPSTable(const std::string &pattern)
{
    int pattern_len = pattern.length();
    std::vector<int> lps_table(pattern_len);
    int table_index = 1;
    int pattern_pos = 0;
    lps_table[0] = -1;
    while (table_index = 0 && pattern[table_index] != pattern[pattern_pos])
    {
        pattern_pos = lps_table[pattern_pos];
        ++table_index;
        ++pattern_pos;
    }

    return lps_table;
}
int KMPSearch(const std::string &text, const std::string &pattern)
{
    int text_len = text.length();
    int pattern_len = pattern.length();
    std::vector lps_table = BuildLPSTable(pattern);
    lps_table[0] = -1;
    int pattern_pos = 0;
    int text_pos = 0;
    while (text_pos < text_len)
    {
        if (text[text_pos] != pattern[pattern_pos])
        {
            pattern_pos = lps_table[pattern_pos];
            if (pattern_pos < 0)
            {
                ++text_pos;
                ++pattern_pos;
            }
            continue;
        }
        ++pattern_pos;
        ++text_pos;
        if (pattern_pos == pattern_len)
        {
            return text_pos - pattern_len;
        }
    }
    return -1;
}
```

## Conclusion

The Knuth-Morris-Pratt algorithm offers a significant improvement over the naive approach by efficiently skipping unnecessary comparisons, making it a powerful tool for text searching in various applications. With a time complexity of _O_(n + m), it’s ideal for scenarios where performance is critical.

Stay tuned for our next post, where we’ll explore the Boyer-Moore algorithm, another efficient text search method that utilizes different heuristics to achieve even faster search times.
