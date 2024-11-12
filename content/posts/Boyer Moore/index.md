+++
author = "Damian Kuraś"
title = "Efficient Text Search Algorithms in C++: Boyer-Moore Algorithm" 
date = 2024-11-12
lastmod = 2024-11-12
description = "Learn how the Boyer-Moore algorithm text search algorithm works and how to implement in C++." 
draft = false
image = "images/bm-text-search-cover.png"
keywords = "Boyer-Moore algorithm, text search in C++"
categories = ["Algorithms"]
tags = ["C++","Search"]
slug = "boyer-moore-text-search"
math = false
+++

## Introduction

The Boyer–Moore algorithm is a powerful tool for text searching, especially useful in applications where efficiency matters. Developed by **Robert S. Boyer** and **J Strother Moore** in 1977, this algorithm significantly outperforms naive string matching techniques by leveraging two key optimization rules that allow it to skip over large portions of text.

## How the Boyer-Moore Algorithm Works

The Boyer-Moore algorithm compares the pattern and text starting from the end of the pattern, which helps in skipping over large sections of text. This is a stark contrast to naive methods that compare characters from the beginning.

![Example showing matching pattern in string starting from last letter of pattern.](./images/matching-backwards.svg)

The Boyer–Moore algorithm uses two main rules to increase its efficiency:

1. **The Bad Character Rule**
2. **The Good Suffix Rule**

These rules help determine how far the search can skip ahead, optimizing performance. With these, the worst-case time complexity is **_O_(n\*m)**, but in most cases, it performs much faster, especially when the pattern is not found, or we need only an index of the first occurrence of a pattern in text, its time complexity is O(n+m).

To apply these rules, we need to create lookup tables based on the pattern.

## The Bad Character rule

The Bad Character Rule helps us decide how far we can shift the pattern to the right after a mismatch, allowing us to skip unnecessary comparisons. Here's how it works:

**Mismatch Detection:**
The algorithm starts comparing from the end of the pattern. When a mismatch occurs, it checks if the mismatched character exists elsewhere in the pattern.

**Finding the Last Occurrence:**
If the mismatched character appears elsewhere in the pattern, the pattern is shifted to align the last occurrence of the mismatched character with the text character. This avoids unnecessary re-checks of already compared characters.

**If No Match is Found:**
If the mismatched character doesn’t appear in the pattern, the algorithm skips past the mismatched character entirely, maximizing the shift distance.

![Example showing shifts after application of the bad character rule](./images/bad-character-rule.svg "Example showing shifts after application of the bad character rule.")

## The Good Suffix rule

The Good Suffix Rule allows us to skip parts of the text when a suffix of the pattern (a portion of characters at the end) matches part of the text but is followed by a mismatch. When this happens, the rule finds where this suffix occurs earlier in the pattern.

**Find Earlier Occurrence:**
If there's an earlier occurrence of the suffix in the pattern, shift the pattern so this occurrence aligns with the matched part in the text.

**Shift to the Longest Prefix:**
If there's no such earlier occurrence, shift the pattern so that the longest prefix of the pattern matches the suffix in the text.

![Example showing shifts after application of good suffix rule](./images/good-suffix-rule.svg "Example showing shifts after application of the good suffix rule.")

## Implementation in c++.

Returns index of first occurrence of pattern in text if not found return -1.

```cpp
const int ALPHABET_LENGTH = 256;
std::vector<int> MakeBadCharShiftArray(const std::string &pattern)
{
    int pattern_len = pattern.length();
    std::vector<int> shift_array(ALPHABET_LENGTH);
    for (int i = 0; i < ALPHABET_LENGTH; i++)
    {
        shift_array[i] = pattern_len;
    }
    for (int i = 0; i < pattern_len; i++)
    {
        shift_array[pattern[i]] = pattern_len - 1 - i;
    }
    return shift_array;
}
// is suffix of word starting from pos a prefix
bool IsSuffixAPrefix(const std::string &word, int pos)
{
    int suffix_len = word.length() - pos;
    for (int i = 0; i < suffix_len; i++)
    {
        if (word[i] != word[pos + i])
        {
            return false;
        }
    }
    return true;
}
// maximum length of the substring ends at pos and is a suffix
int SuffixLen(const std::string &word, int pos)
{
    int len = 0;
    while ((word[pos - len] == word[word.length() - 1 - len]) && (len <= pos))
    {
        ++len;
    }
    return len;
}
std::vector<int> MakeGoodSuffixShiftArray(const std::string &pattern)
{
    int pattern_len = pattern.length();
    int last_prefix_pos = 1;
    std::vector<int> shift_array(pattern_len);
    for (int i = pattern_len - 1; i >= 0; i--)
    {
        if (IsSuffixAPrefix(pattern, i + 1))
        {
            last_prefix_pos = i + 1;
        }
        shift_array[i] = last_prefix_pos + (pattern_len - 1 - i);
    }
    for (int i = 0; i < pattern_len - 1; i++)
    {
        int suffix_len = SuffixLen(pattern, i);
        if (pattern[i - suffix_len] != pattern[pattern_len - 1 - suffix_len])
        {
            shift_array[pattern_len - 1 - suffix_len] = pattern_len - 1 - i + suffix_len;
        }
    }
    return shift_array;
}

int BMSearch(const std::string &text, const std::string &pattern)
{
    int text_len = text.length();
    int pattern_len = pattern.length();
    std::vector shift_bad_char = MakeBadCharShiftArray(pattern);
    std::vector shift_good_suffix = MakeGoodSuffixShiftArray(pattern);
    if (pattern_len == 0)
    {
        return -1;
    }
    int text_pos = pattern_len - 1;
    while (text_pos < text_len)
    {
        int patter_pos = pattern_len - 1;
        while (patter_pos >= 0 && text[text_pos] == pattern[patter_pos])
        {
            --patter_pos;
            --text_pos;
        }
        if (patter_pos < 0)
        {
            return text_pos + 1;
        }
        int shift = std::max(shift_bad_char[text[text_pos]], shift_good_suffix[patter_pos]);
        text_pos += shift;
    }
    return -1;
}
```

## Conclusion: Why the Boyer-Moore Algorithm is Essential

The **Boyer-Moore algorithm** is widely used in C++ libraries (since C++17) and is also featured in popular tools like GNU grep. Chances are, you've used it yourself when performing text searches on your computer. Its combination of the **Bad Character** and **Good Suffix** rules makes it an optimal choice for many text searching tasks.

You can check my previous post about Knuth-Morris-Pratt algorithm [here]({{<ref "/posts/kmp text search">}} )
