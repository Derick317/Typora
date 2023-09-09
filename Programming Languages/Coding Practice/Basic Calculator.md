# Basic Calculator

## Description

Given a string `s` representing a valid expression, implement a basic calculator to evaluate it, and return *the result of the evaluation*.

`s` consists of digits, `'+'`, `'-'`, `'('`, `')'`, and `' '` and represents a valid expression. `'+'` is **not** used as a unary operation (i.e., `"+1"` and `"+(2 + 3)"` is invalid). `'-'` could be used as a unary operation (i.e., `"-1"` and `"-(2 + 3)"` is valid).

## Algorithm

Although we can convert this infix expression into postfix expression, it is not necessary because there are no `'*'` or `'/'`. Nevertheless, a stack is required.

We could see first how to evaluate an expression without `'('` or `')'`, such as `"-1"` and `"2 + 3"`. The later one can be seen as `"+ 2 + 3"`. In this way, the number of operators equals the number of operands. We can start with `0`, if we meet `"+"`, we add the next number to the result; otherwise, subtract the next number from the result. 

Now, let's consider parentheses. The expression in a pair of parentheses should be considered as a number. When evaluate this sub-expression, we need to store current result and its sign. The best data structure is exactly stack.

```c++
struct Item
{
    int result;
    bool plus;
};

int calculate(string s)
{
    stack<Item> stk;
    int start = 0;
    int result = 0;
    bool plus = true;
    while (start < s.size())
    {
        start = s.find_first_not_of(' ', start);
        if (start == string::npos)
            break;
        if ('0' <= s[start] && s[start] <= '9')
        {
            int end = s.find_first_of("()+- ", start);
            if (plus)
                result += stoi(s.substr(start, end - start));
            else
                result -= stoi(s.substr(start, end - start));
            start = end;
        }
        else if ('+' == s[start])
        {
            ++start;
            plus = true;
        }
        else if ('-' == s[start])
        {
            ++start;
            plus = false;
        }
        else if ('(' == s[start])
        {
            stk.push({result, plus});
            result = 0;
            plus = true;
            ++start;
        }
        else
        {
            if (stk.top().plus)
                result += stk.top().result;
            else
                result = stk.top().result - result;
            stk.pop();
            ++start;
        }
    }
    return result;
}
```

