+++
title = "About Me"
date = 2019-09-06T19:20:50+08:00
path = "about"
template = "page.html"
+++

# Elvis Wang

Email: <mail@wangbo.im>

# Live in Code

## Java version

```java
public final class Person {
    private final String name_;
    private final String gender_;

    private String location_;
    private String from_;
    private String currentCompany_;
    private String email_;

    private volatile boolean isPersonAlive;

    public Person(final String name, final String gender) {
        this.name_ = name;
        this.gender_ = gender;

        this.location_ = "";
        this.from_ = "";
        this.currentCompany_ = "";
        this.email_ = "";

        this.isPersonAlive = true;
    }

    public Person location(final String val) {
        this.location_ = val;
        return this;
    }

    public Person from(final String val) {
        this.from_ = val;
        return this;
    }

    public Person workFor(final String val) {
        this.currentCompany_ = val;
        return this;
    }

    public Person email(final String val) {
        this.email_ = val;
        return this;
    }

    public void live() {
        // Struggle to live
    }

    public boolean isAlive() {
        return isPersonAlive;
    }

    public static void main(String [] _args) {
        final Person elvisWang = new Person("Elvis Wang", "Male");

        elvisWang.workFor("58.com")
                 .location("Beijing, Beijing, P.R. China")
                 .from("Anqing, Anhui, P.R. China")
                 .email("mail@wangbo.im");

        while (elvisWang.isAlive()) {
            elvisWang.live();
        }
    }
}
```

## C++ version

```c++
class Person {
private:
    string name_;
    string gender_;

    string location_;
    string from_;
    string currentCompany_;
    string email_;

    volatile bool isPersonAlive;

public:
    Person(string &n, string &g) {
        name_ = n;
        gender_ = g;

        isPersonAlive = true;
    }
    ~Person() {
        // todo
    }

public:
    Person & location(const string &val){
        this->location_ = val;
        return *this;
    }

    Person & from(const string &val){
        this->from_ = val;
        return *this;
    }

    Person & workFor(const string &val){
        this->currentCompany_ = val;
        return *this;
    }

    Person & email(const string &val){
        this->email_ = val;
        return *this;
    }

    void live() {
        // Struggle to live
    }

    bool isAlive() {
        return isPersonAlive;
    }

private:
    Person(const Person &);
    Person &operator=(const Person &);
}

int main(int argc, char *argv[]) {
    Person elvisWang("Elvis Wang", "Male");

    elvisWang.workFor("58.com")
             .location("Beijing, Beijing, P.R. China")
             .from("Anqing, Anhui, P.R. China")
             .email("mail@wangbo.im");

    while (elvisWang.isAlive()) {
        elvisWang.live();
    }
}
```
