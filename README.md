#include <iostream>
#include <string>
#include <vector>
#include <stdexcept>
using namespace std;

class BigInt {
    string digits;

public:
    // Constructors
    BigInt(unsigned long long n = 0);
    BigInt(const string &);
    BigInt(const char *);
    BigInt(const BigInt &);

    // Helper Functions
    friend bool Null(const BigInt &);
    friend int Length(const BigInt &);
    int operator[](const int) const;

    // Operator Overloading
    BigInt &operator=(const BigInt &);

    BigInt &operator++();
    BigInt operator++(int);
    BigInt &operator--();
    BigInt operator--(int);

    friend BigInt &operator+=(BigInt &, const BigInt &);
    friend BigInt operator+(const BigInt &, const BigInt &);
    friend BigInt operator-(const BigInt &, const BigInt &);
    friend BigInt &operator-=(BigInt &, const BigInt &);

    friend bool operator==(const BigInt &, const BigInt &);
    friend bool operator!=(const BigInt &, const BigInt &);
    friend bool operator>(const BigInt &, const BigInt &);
    friend bool operator>=(const BigInt &, const BigInt &);
    friend bool operator<(const BigInt &, const BigInt &);
    friend bool operator<=(const BigInt &, const BigInt &);

    friend BigInt &operator*=(BigInt &, const BigInt &);
    friend BigInt operator*(const BigInt &, const BigInt &);
    friend BigInt &operator/=(BigInt &, const BigInt &);
    friend BigInt operator/(const BigInt &, const BigInt &);

    friend BigInt operator%(const BigInt &, const BigInt &);
    friend BigInt &operator%=(BigInt &, const BigInt &);

    friend ostream &operator<<(ostream &, const BigInt &);
    friend istream &operator>>(istream &, BigInt &);
};

BigInt::BigInt(const string &s) {
    if (s.empty() || s == "0") {
        digits = "0";
        return;
    }
    for (int i = s.size() - 1; i >= 0; --i) {
        if (!isdigit(s[i]))
            throw invalid_argument("Invalid digit in string");
        digits.push_back(s[i] - '0');
    }
}

BigInt::BigInt(unsigned long long nr) {
    if (nr == 0) {
        digits.push_back(0);
    } else {
        while (nr != 0) {
            digits.push_back(nr % 10);
            nr /= 10;
        }
    }
}

BigInt::BigInt(const char *s) {
    string str(s);
    *this = BigInt(str);
}

BigInt::BigInt(const BigInt &a) : digits(a.digits) {}

bool Null(const BigInt &a) {
    return a.digits.size() == 1 && a.digits[0] == 0;
}

int Length(const BigInt &a) {
    return a.digits.size();
}

int BigInt::operator[](const int index) const {
    if (index < 0 || index >= digits.size())
        throw out_of_range("Index out of range");
    return digits[index];
}

BigInt &BigInt::operator=(const BigInt &a) {
    if (this != &a) {
        digits = a.digits;
    }
    return *this;
}

BigInt &BigInt::operator++() {
    int i = 0, n = digits.size();
    while (i < n && digits[i] == 9) {
        digits[i] = 0;
        ++i;
    }
    if (i == n)
        digits.push_back(1);
    else
        ++digits[i];
    return *this;
}

BigInt BigInt::operator++(int) {
    BigInt temp = *this;
    ++(*this);
    return temp;
}

BigInt &BigInt::operator--() {
    if (digits.size() == 1 && digits[0] == 0)
        throw underflow_error("Underflow error");
    int i = 0;
    while (digits[i] == 0) {
        digits[i] = 9;
        ++i;
    }
    --digits[i];
    if (digits.size() > 1 && digits.back() == 0)
        digits.pop_back();
    return *this;
}

BigInt BigInt::operator--(int) {
    BigInt temp = *this;
    --(*this);
    return temp;
}

BigInt &operator+=(BigInt &a, const BigInt &b) {
    int carry = 0, i;
    int n = a.digits.size(), m = b.digits.size();
    if (m > n) a.digits.resize(m, 0);
    for (i = 0; i < n || i < m || carry; ++i) {
        if (i == a.digits.size()) a.digits.push_back(0);
        a.digits[i] += carry + (i < m ? b.digits[i] : 0);
        carry = a.digits[i] >= 10;
        if (carry) a.digits[i] -= 10;
    }
    return a;
}

BigInt operator+(const BigInt &a, const BigInt &b) {
    BigInt temp = a;
    temp += b;
    return temp;
}

BigInt &operator-=(BigInt &a, const BigInt &b) {
    if (a < b)
        throw underflow_error("Underflow error");
    int borrow = 0, i;
    for (i = 0; i < a.digits.size(); ++i) {
        a.digits[i] -= (i < b.digits.size() ? b.digits[i] : 0) + borrow;
        borrow = a.digits[i] < 0;
        if (borrow) a.digits[i] += 10;
    }
    while (a.digits.size() > 1 && a.digits.back() == 0)
        a.digits.pop_back();
    return a;
}

BigInt operator-(const BigInt &a, const BigInt &b) {
    BigInt temp = a;
    temp -= b;
    return temp;
}

bool operator==(const BigInt &a, const BigInt &b) {
    return a.digits == b.digits;
}

bool operator!=(const BigInt &a, const BigInt &b) {
    return !(a == b);
}

bool operator<(const BigInt &a, const BigInt &b) {
    if (a.digits.size() != b.digits.size())
        return a.digits.size() < b.digits.size();
    for (int i = a.digits.size() - 1; i >= 0; --i) {
        if (a.digits[i] != b.digits[i])
            return a.digits[i] < b.digits[i];
    }
    return false;
}

bool operator>(const BigInt &a, const BigInt &b) {
    return b < a;
}

bool operator<=(const BigInt &a, const BigInt &b) {
    return !(a > b);
}

bool operator>=(const BigInt &a, const BigInt &b) {
    return !(a < b);
}

BigInt &operator*=(BigInt &a, const BigInt &b) {
    if (Null(a) || Null(b)) {
        a = BigInt();
        return a;
    }
    vector<int> v(a.digits.size() + b.digits.size(), 0);
    for (int i = 0; i < a.digits.size(); ++i) {
        for (int j = 0; j < b.digits.size(); ++j) {
            v[i + j] += a.digits[i] * b.digits[j];
        }
    }
    a.digits.resize(v.size());
    int carry = 0;
    for (int i = 0; i < v.size(); ++i) {
        v[i] += carry;
        a.digits[i] = v[i] % 10;
        carry = v[i] / 10;
    }
    while (a.digits.size() > 1 && a.digits.back() == 0)
        a.digits.pop_back();
    return a;
}

BigInt operator*(const BigInt &a, const BigInt &b) {
    BigInt temp = a;
    temp *= b;
    return temp;
}

BigInt &operator/=(BigInt &a, const BigInt &b) {
    if (Null(b)) throw invalid_argument("Division by zero");
    BigInt result, current;
    for (int i = a.digits.size() - 1; i >= 0; --i) {
        current.digits.insert(current.digits.begin(), a.digits[i]);
        int x = 0, left = 0, right = 10;
        while (left <= right) {
            int mid = (left + right) / 2;
            BigInt temp = b * BigInt(mid);
            if (temp <= current) {
                x = mid;
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        result.digits.insert(result.digits.begin(), x);
        current -= b * BigInt(x);
    }
    while (result.digits.size() > 1 && result.digits.back() == 0)
        result.digits.pop_back();
    a.digits = result.digits;
    return a;
}

BigInt operator/(const BigInt &a, const BigInt &b) {
    BigInt temp = a;
    temp /= b;
    return temp;
}

BigInt &operator%=(BigInt &a, const BigInt &b) {
    if (Null(b)) throw invalid_argument("Division by zero");
    BigInt current;
    for (int i = a.digits.size() - 1; i >= 0; --i) {
        current.digits.insert(current.digits.begin(), a.digits[i]);
        int x = 0, left = 0, right = 10;
        while (left <= right) {
            int mid = (left + right) / 2;
            BigInt temp = b * BigInt(mid);
            if (temp <= current) {
                x = mid;
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        current -= b * BigInt(x);
    }
    while (current.digits.size() > 1 && current.digits.back() == 0)
        current.digits.pop_back();
    a.digits = current.digits;
    return a;
}

BigInt operator%(const BigInt &a, const BigInt &b) {
    BigInt temp = a;
    temp %= b;
    return temp;
}

ostream &operator<<(ostream &out, const BigInt &a) {
    for (int i = a.digits.size() - 1; i >= 0; --i)
        out << (short)a.digits[i];
    return out;
}

istream &operator>>(istream &in, BigInt &a) {
    string s;
    in >> s;
    a = BigInt(s);
    return in;
}

int main() {
    BigInt a, b;
    cout << "Enter two big integers: ";
    cin >> a >> b;

    cout << "a + b = " << a + b << endl;
    cout << "a - b = " << a - b << endl;
    cout << "a * b = " << a * b << endl;
    cout << "a / b = " << a / b << endl;
    cout << "a % b = " << a % b << endl;

    return 0;
}
