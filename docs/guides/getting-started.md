# Getting started

## Basic Usage

After [installing the shard](../installation.md), you will need to follow three
steps to start using Blueprint:

1. Require `"blueprint/html"`
1. Include `Blueprint::HTML` module
1. Define a `#blueprint` method to write the HTML structure inside.

=== ":simple-crystal: ExamplePage"

    ```crystal
    require "blueprint/html"

    class ExamplePage
      include Blueprint::HTML

      private def blueprint
        h1 { "Hello World" }
      end
    end
    ```

Your view class is now defined, to render it it's necessary to instantiate it
and call `#to_html` method.

=== ":simple-crystal: Main"

    ```crystal
    example = Example.new
    example.to_html # => "<h1>Hello World</h1>
    ```

## Blueprints Are Just POCOs
You are using just Plain Old Crystal Objects (POCOs). The `Blueprint::HTML` just
add helper methods to handle the HTML building stuff and your class can have its
own logic. For example, if you want pass data to your view object:

=== ":simple-crystal: ProfilePage"

    ```crystal
    class ProfilePage
      include Blueprint::HTML

      def initialize(@user : User); end

      private def blueprint
        h1 { @user.name }

        admin_badge if admin?
      end

      private def admin_badge
        img src: "admin-badge.png"
      end

      private def admin?
        @user.role == "admin"
      end
    end
    ```

=== ":simple-crystal: Main"

    ```crystal
    user = User.new(name: "Jane Doe", role: "admin")
    page = ProfilePage.new(user: user)
    page.to_thml
    ```

=== ":simple-html5: Output"

    ```html
    <h1>Jane Doe</h1>
    <img src="admin-badge.png">
    ```

It is possible even use structs instead of classes:

=== ":simple-crystal: Main"

    ```crystal
    struct Card
      include Blueprint::HTML

      private def blueprint
        div(class: "card") { "Hello" }
      end
    end

    # or using `record` macro
    record Alert, message : String do
      include Blueprint::HTML

      private def blueprint
        div(class: "alert") { @message }
      end
    end

    card = Card.new
    card.to_html # => <div class="card">Hello</div>

    alert = Alert.new(message: "Hello")
    alert.to_html # => <div class="alert">Hello</div>
    ```

## Passing Content

If you want to pass content when rendering a blueprint, you should define the
`#blueprint(&)` method and yield the block.

=== ":simple-crystal: Alert"

    ```crystal
    class Alert
      include Blueprint::HTML

      private def blueprint(&)
        div class: "alert" do
          yield
        end
      end
    end
    ```

=== ":simple-crystal: Main"

    ```crystal
    alert = Alert.new
    alert.to_html { "Hello World" }
    ```

=== ":simple-html5: Output"

    ```html
    <div class="alert">
      Hello World
    </div>
    ```

## Code style

If you are new to Crystal, know that blocks can be passed using `do ... end` or
`{ ... }`. All of these are equivalent:

```crystal

# With `do ... end` blocks
private def blueprint
  html lang: "pt-BR" do
    head do
      title do
        "Blueprint"
      end
    end

    body do
      h1 class: "heading" do
        "Hello World"
      end
    end
  end
end

# With `{ ... }` blocks
private def blueprint
  html lang: "pt-BR" {
    head {
      title {
        "Blueprint"
      }
    }

    body {
      h1 class: "heading" {
        "Hello World"
      }
    }
  }
end
```

It's a personal or team preference, you can stick to one style or mix two
styles (eg. Use `do ... end` for multiline blocks and `{ ... }` for inline
blocks). But note that in Crystal the difference between using `do ... end` and
`{ ... }` is that `do ... end` binds to the left-most call, while `{ ... }`
binds to the right-most call:

With `do ... end` blocks:
```crystal
private def blueprint
  render Alert.new do
    "Hello World"
  end
end

# The above is same as
private def blueprint
  render(Alert.new) do
    "Hello World"
  end
end
```

While using `{ ... }` blocks:
```crystal
private def blueprint
  render Alert.new {
    "Hello World"
  }
end

# The above is same as
private def blueprint
  render(Alert.new {
    "Hello World"
  })
end
```

The above example will raise an error `Error: 'Alert.new' is not expected to
be invoked with a block, but a block was given`, to fix this you need to add
parentheses to `render` call when using with `{ ... }`.

```crystal
private def blueprint
  render(Alert.new) {
    "Hello World"
  }
end
```
