Handy Snippets
==============

Java
----

Change code just by inheritance parametrization.

~~~ { .java }
// Build & run:
// javac -Xlint:all Example.java && java Example
import java.lang.Exception;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Method;

public class Example {

    public static void main(String args[]){
        new SimpleTest().test();
        new SimpleTest2().test();
    }
}

class State {
    public String url;

    public State() {
        this("localhost");
    }

    public State(String url) {
        this.url = url;
    }
}

abstract class BasePage {
    protected final State state;

    public BasePage(State state) {
        this.state = state;
    }

    public static BasePage magic() throws Exception {
        throw new Exception("unimplemented");
    };
}

class SimplePage extends BasePage {
    public SimplePage(State state) {
        super(state);
    }

    // @Override
    public static SimplePage magic(State state) {
        state.url = "url://base/page";
        return new SimplePage(state);
    }
}

class SimplePage2 extends BasePage {
    public SimplePage2(State state) {
        super(state);
    }

    // @Override
    public static SimplePage2 magic(State state) {
        state.url = "url://base/page-2";
        return new SimplePage2(state);
    }
}

class WithPageTest<T extends BasePage> {
    protected T inner;
    protected State state;


    public Class<T> getParamClass() throws Throwable {
        ParameterizedType superclass = (ParameterizedType) getClass().getGenericSuperclass();
        @SuppressWarnings("unchecked")
        Class<T> clazz = (Class<T>) superclass.getActualTypeArguments()[0];
        return clazz;
    }

    public WithPageTest() {
        this.state = new State();

        try {
            Class<T> c = getParamClass();
            Method m = c.getMethod("magic", State.class);
            @SuppressWarnings("unchecked")
            T inner = (T)(m.invoke(null, state));
            this.inner = inner;
        } catch (Throwable e) {
        }
    }
}

class SimpleTest extends WithPageTest<SimplePage> {

    public void test() {
        System.out.println("url: " + this.inner.state.url);
    }
}

class SimpleTest2 extends WithPageTest<SimplePage2> {

    public void test() {
        System.out.println("url: " + this.inner.state.url);
    }
}
~~~

Bash
----

Remove imports from **Haskell** code (so number of lines can be counted without imports).

~~~ { .bash }
find . -name '*.hs' -print0 \
| xargs -n1 -0 sed -i -n '/^\s*$/d;H;${x;s/^\n//;s/\nimport[^\n]*\(\n [^\n]*\)*//g;p}'
~~~

Generate `Portability` Haddock header from `LANGUAGE` pragmas for **Haskell** code.
(Does not work well when `LANGUAGE` pragmas are included only conditionally via `CPP`.)

~~~ { .bash }
cat "${HS_FILE}" \
| sed -n 's/^{-# LANGUAGE \(.*\) #-}$/\1/p' \
| tr ',' '\n' | tr -d ' ' \
| sort -u | tr '\n' ',' \
| sed 's/\s*,\s*/, /g;s/, $//;s/\(.\{,61\}\(, \|$\)\)/--               \1\n/g;s/^-- \{13\}/-- Portability:/;s/, \n/,\n/g'
~~~
