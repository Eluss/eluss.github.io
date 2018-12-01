---
layout: post
title: Expressible by array literal
category: blog
comments: true
tags: [swift, protocols]
header: /assets/images/expressible-by-array-literal/expressible-by-array-literal.jpeg  
excerpt: I recently noticed a pretty nice mechanism which may come in handy in some situations - Expressible by array literal.
---

I recently noticed a pretty nice mechanism which may come in handy in some situations ;)

Let's imagine we have a `Team` structure that needs a configuration in order to be initialized. 
If we want to get some information about the team, we will simply call `team.info()`.
{% highlight swift %}
typealias Player = String
struct Team {

    let configuration: [Player]

    init(configuration: [Player]) {
        self.configuration = configuration
    }

    func info() {
        let names = configuration.reduce(into: "") { 
            (result: inout String, player: Player) in
            if result.isEmpty {
                result += player
            } else {
                result += ", \(player)"
            }
        }
        print("The team structure is: \(names)")
    }
}

let team = Team(configuration: ["Foo", "Bar"])
print(team.info()) // The team structure is: Foo, Bar
{% endhighlight %}

But at some point it turns out, that we would like to create another team object, however its configuratin is unknown.
We probably shouldn't initialize it with an empty array of players (`let team = Team(configuration: [])`) as this could be reserved for a team without any players at all. What can we do about it?

## ExpressibleByArrayLiteral

As we already have an initializer that we use in many different places which we don't want to change, we can use a protocol called `ExpressibleByArrayLiteral`. This will allow us to create an enum like: 

{% highlight swift %}

enum Configuration: ExpressibleByArrayLiteral {

    case players([Player])
    case unknown

    typealias ArrayLiteralElement = Player

    init(arrayLiteral elements: ArrayLiteralElement...) {
        self = .players(elements)
    }

}
{% endhighlight %}

Now we can replace the constructor of a `Team` object with  
`init(configuration: Configuration)` and use it this way:  
`let unknownTeam = Team(configuration: .unknown)`. Keep in mind, that we do not need to change the code that created the old team  
`let myTeam = Team(configuration: ["Foo", "Bar"])`, because an array will be translated to our enum as `case players([Player])`. 

{% highlight swift %}
struct Team {

    let configuration: Configuration

    init(configuration: Configuration) {
        self.configuration = configuration
    }

    func info() {
        switch configuration {
        case .players(let players):
            let names = players.reduce(into: "") { 
                (result: inout String, player: Player) in
                result += player + " "
            }
            print("The team structure is: \(names)")
        case .unknown: print("Unknown team configuration")
        }
    }
}

let myTeam = Team(configuration: ["Foo", "Bar"])
let unknownTeam = Team(configuration: .unknown)

print(myTeam.info()) // The team structure is: Foo, Bar
print(unknownTeam.info()) // Unknown team configuration

{% endhighlight %}

If you find this interesting, then take a look at the [official docs](https://developer.apple.com/documentation/swift/expressiblebyarrayliteral) or `ExpressibleByStringLiteral`, `ExpressibleByNilLiteral`, `ExpressibleByBooleanLiteral`, `ExpressibleByFloatLiteral` and other `ExpressibleBy*Literal` protocols ;)