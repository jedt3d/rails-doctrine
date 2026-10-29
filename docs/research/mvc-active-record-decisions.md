# Why Rails reads MVC the way it does, and why it chose Active Record over Data Mapper

Research notes compiled 2026-10-29 from primary sources only. Every quotation below is verbatim from the cited URL as fetched on that date (HTML converted to text; typographic quotes preserved where the source used them). Where a source could not be fetched, or a quote could only be partially recovered, that is stated explicitly in the final section rather than papered over.

## Summary

- Rails' own statement of intent is **The Rails Doctrine** (DHH, rubyonrails.org/doctrine). It names nine pillars and, under "No one paradigm", gives the clearest first-party justification for Active Record: "We're mixing the logic needed for interfacing with the database directly with the business domain and logic. Such conflation of boundaries! Yes, because it proved to be the practical way to skin a web-app cat that virtually always talks to a database of some sort to persist the state of the domain model."
- The official guides describe MVC plainly: the model "Manages the data in your application", the view "Handles rendering responses", the controller "Handles user interactions and the logic for each request" (Getting Started), and "Active Record is part of the M in MVC - the model - which is the layer of the system responsible for representing data and business logic" (Active Record Basics). Active Model and plain Ruby objects are also "part of the M in MVC".
- Rails explicitly cites Fowler's PoEAA definition of Active Record. Fowler's catalog defines Data Mapper as the opposite choice: "A layer of mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself."
- The Rails team acknowledges the trade-offs itself: conventions have "peril", the multi-paradigm approach costs "conceptual overhead", concerns "can indeed be used" to "stuff their clutter in", the Doctrine says "We don't try to shield you that much from having to learn ... SQL", and Manrubia concedes "Active Record requires you to control the database schema to leverage what it offers fully" and that "you can use it to write messy and unmaintainable code".
- The "sharp knives" the framework ships to manage those trade-offs, each with a first-party doc comment: `ActiveSupport::Concern`, `normalizes`, `delegated_type` (DHH's long comment explains why over abstract classes and STI), `strict_loading` / `strict_loading_by_default`, `ActiveModel::API` / `ActiveModel::Model` for non-persisted models, and (8.1) `has_many ..., deprecated: true`.
- 37signals' published position (Manrubia, dev.37signals.com) is that business logic lives in "rich domain models" (both Active Records and POROs) called directly from controllers and jobs, organized with concerns, delegating to "additional systems of objects" when complexity justifies it; services are not "first-class architectural artifacts".
- Authentication: DHH opened issue #50446 (Dec 2023) stating the intent "to teach Rails developers how to use the basic blocks"; PR #52328 (merged 2024-07-16) shipped it, explicitly "not intended to be an all-singing, all-dancing answer". The generated `Authentication` concern plus `Current` is the mechanism; the Security guide's authorization advice is to scope queries through the current user: `@project = @current_user.projects.find(params[:id])`.
- Rails 8.0 (Nov 2024): Kamal 2, Thruster, Solid Cable/Cache/Queue, Propshaft default, authentication generator, `params.expect`. Rails 8.1 (Oct 2025): Active Job Continuations, Structured Event Reporting, Local CI, Markdown rendering, `credentials:fetch`, deprecated associations, registry-free Kamal.
- Core team (first names as published): David, Jeremy, Aaron, Xavier, Rafael, Guillermo, Carlos, Matthew, Eileen, Ryuta, Jean, John, Mike. Committers: Akira, Eugene, Dima, Gannon, Hartley, Jonathan, Yasuo.

---

## 1. Rails Doctrine pillars (quoted)

Source: David Heinemeier Hansson, "The Rails Doctrine", https://rubyonrails.org/doctrine

Framing:

> "Ruby on Rails' phenomenal rise to prominence owed much of its lift-off to novel technology and timing. But technological advantages erode over time, and good timing doesn't sustain movements alone over the long term. So a broader explanation of how Rails has continued to not only stay relevant but to grow its impact and community is needed. I propose that the enduring enabler has been and remains its controversial doctrine."

> "With all that ado, here follows the nine most important pillars of The Rails Doctrine, as perceived by yours truly:"

The nine pillars, exact titles as listed:

1. Optimize for programmer happiness
2. Convention over Configuration
3. The menu is omakase
4. No one paradigm
5. Exalt beautiful code
6. Provide sharp knives
7. Value integrated systems
8. Progress over stability
9. Push up a big tent

### 1.1 "Convention over Configuration"

> "One of the early productivity mottos of Rails went: 'You're not a beautiful and unique snowflake'. It postulated that by giving up vain individuality, you can leapfrog the toils of mundane decisions, and make faster progress in areas that really matter."

> "Who cares what format your database primary keys are described by? Does it really matter whether it's 'id', 'postId', 'posts_id', or 'pid'? Is this a decision that's worthy of recurrent deliberation? No."

> "Not only does the transfer of configuration to convention free us from deliberation, it also provides a lush field to grow deeper abstractions. If we can depend on a Person class mapping to people table, we can use that same inflection to map an association declared as has_many :people to look for a Person class. The power of good conventions is that they pay dividends across a wide spectrum of use."

> "But beyond the productivity gains for experts, conventions also lower the barriers of entry for beginners. There are so many conventions in Rails that a beginner doesn't even need to know about, but can just benefit from in ignorance. It's possible to create great applications without knowing why everything is the way it is."

Acknowledged trade-off, in the same section:

> "As with anything, though, the power of convention isn't without peril. When Rails makes it so trivial to do so much, it is easy to think every aspect of an application can be formed by precut templates. But most applications worth building have some elements that are unique in some way. It may only be 5% or 1%, but it's there."

> "The hard part is knowing when to stray from convention. When are the deviating particulars grave enough to warrant an excursion? I contend that most impulses to be a beautiful and unique snowflake are ill considered, and that the cost of going off the Rails is under appreciated, but just enough of them won't be that you need to examine all of them carefully."

### 1.2 "The menu is omakase"

> "How do you know what to order in a restaurant when you don't know what's good? Well, if you let the chef choose, you can probably assume a good meal, even before you know what 'good' is. That is omakase. A way to eat well that requires you neither be an expert in the cuisine nor blessed with blind luck at picking in the dark."

> "For programming, the benefits of this practice, letting others assemble your stack, is similar to those we derive from Convention over Configuration, but at a higher level. Where CoC is occupied with how we best use individual frameworks, omakase is concerned with which frameworks, and how they fit together."

> "So with Rails we decided to diminish one good, a programmer's individual privilege to choose each tool in their box, for a greater one: A better tool box for all. The dividends are legion:"

> "There's safety in numbers: When most people are using Rails in the same default ways, we have a shared experience. This common ground makes it much easier to teach and help people. It lays a foundation for debate on approach. We all watched the same show last night at 7, so we can talk about it the next day. It fosters a stronger sense of community."

> "People are perfecting the same, basic tool box: As a full-stack framework, Rails has a lot of moving parts, and how those work together is as important as what they do in isolation. Much of the pain in software comes not from the individual components, but from their interaction. When we all work on alleviating shared pain from components that are configured and fail in the same ways, then we all experience less pain."

> "Substitutions are still possible, but not required: While Rails is an omakase stack, it still allows you to replace certain frameworks or libraries with alternatives. It just doesn't require you to. Which means you can delay those decisions until you've developed a clear, personal palate that may prefer the occasional difference."

### 1.3 "No one paradigm" (the section that justifies Active Record and concerns)

> "Rails isn't like that. It isn't a single, perfect cut of cloth. It's a quilt. A composite of many different ideas and even paradigms. Many that would usually be seen in conflict, if contrasted alone and one by one. But that's not what we're trying to do. It isn't a single championship of superior ideas where a sole winner must be declared."

On views (the "V" in Rails' MVC reading):

> "Take the templates we build the view in our Rails MVC pie with. By default, all the helpers that allow us to extract code from these templates are just a big pot of functions! It's a single namespace even. Oh the shock and the horror, it's like PHP soup!"

> "This doesn't mean we don't occasionally want to reach for something more object-oriented when building views. The concept of Presenters, where we wrap many methods that are interdependent with each other and the data below it, can occasionally be the perfect antidote to a soup of methods turned sour by dependencies. But it's generally proved to be the rare rather than common fit."

On models (the "M"):

> "In comparison, we generally treat the model in our MVC layer cake as the prime bastion of object-oriented goodness. Finding just the right names for objects, increasing the coherence, and lowering the coupling is the fun of domain modeling. It's a very different layer from the view, so we take a different approach."

> "But even here we don't subscribe to single-paradigm dogma. Rails concerns, the specialization of Ruby's mixins, are often used to give the individual models a very wide surface area. This fits well with the Active Record pattern by giving the concerned methods direct access to the data and storage they interact with."

The Active Record justification:

> "Even the very foundation of the Active Record framework offends some purists. We're mixing the logic needed for interfacing with the database directly with the business domain and logic. Such conflation of boundaries! Yes, because it proved to be the practical way to skin a web-app cat that virtually always talks to a database of some sort to persist the state of the domain model."

> "To be so ideologically flexible is what enables Rails to tackle such a wide array of problems. Most individual paradigms do very well within a certain slice of the problem space, but become awkward or rigid when applied beyond its natural sphere of comfort. By applying many overlapping paradigms, we cover the flanks and guard the rear. The final framework is far stronger and more capable than any individual paradigm would have allowed it to be."

Acknowledged trade-off:

> "Now, the cost of this polyamorous relationship with the many paradigms of programming is conceptual overhead. It's not enough to just know object-oriented programming to have a good time with Rails. It's preferable to be well served with procedural and functional experiences as well."

> "This applies to the many sub-languages of Rails as well. We don't try to shield you that much from having to learn, say, JavaScript for the view or SQL for the occasional complicated query. At least not to reach the peaks of possibilities."

> "The way to alleviate some of that learning burden is to simply just make it easy to get started, make something of real value, before you understand every single aspect of the framework. We have a rush to Hello World for this reason. Your table already prepared and an appetizer served."

### 1.4 "Provide sharp knives"

> "Ruby includes a lot of sharp knives in its drawer of features. Not by accident, but by design. The most famous is monkey patching: The power to change existing classes and methods."

> "Nothing, is the answer. There's nothing programmatically in Ruby to stop you using its sharp knives to cut ties with reason. We enforce such good senses by convention, by nudges, and through education. Not by banning sharp knives from the kitchen and insisting everyone use spoons to slice tomatoes."

> "That brings us to Rails. The knives provided by the framework are not nearly as sharp as those offered with the language, but some are still plenty keen to cut. We will make no apologies for offering such tools as part of the kit. In fact, we should celebrate having enough faith in the aspirations of our fellow programmers to dare trust them."

On concerns specifically (and the acknowledged risk):

> "Plenty of features in Rails have been contested over time as being 'too much freedom'. But one example that's currently in vogue is the feature of concerns. This is a thin layer of syntactic sugar around Ruby's built-in feature of modules and is designed to allow a single class to encapsulate multiple related but independently understood concerns (hence the name)."

> "The charge is that concerns provide programmers prone to bloat their objects with a whole new set of drawers to stuff their clutter in. And that's true. Concerns can indeed be used like that."

> "But the grand fallacy is thinking that by not providing a feature like concerns, which when used by even mildly capable hands allows an eloquent partial separation of concepts, we'd put programmers on the path to architectural bliss. If you can't be trusted to keep the kitchen sink out of your overstuffed concerns, you're probably not going to end up with a shining beacon of elegance otherwise."

> "That does not abdicate a responsibility to help get them there. The language and the framework should be patient tutors willing to help and guide anyone to experthood. While recognizing that the only reliable course there goes through the land of mistakes: Tools used wrong, a bit of blood, sweat, and perhaps even some tears. There simply is no other way."

### 1.5 "Value integrated systems"

> "Rails can be used in many contexts, but its first love is the making of integrated systems: Majestic monoliths! A whole system that addresses an entire problem. This means Rails is concerned with everything from the front-end JavaScript needed to make live updates to how the database is migrated from one version to another in production."

> "That's a very broad scope, as we've discussed, but no broader than to be realistic to understand for a single person. Rails specifically seeks to equip generalist individuals to make these full systems. Its purpose is not to segregate specialists into small niches and then require whole teams of such in order to build anything of enduring value."

> "It is this focus on empowering the individual that points to the integrated system. It's in the integrated system we can cut out many needless abstractions, reduce the duplication between layers (like templates on both the server and the client), and, above all, avoid distributing our system before we absolutely, positively have to."

> "Much of the complication in systems development comes from introducing new boundaries between the elements that restrict how you make calls between A and B. Method calls between objects is far simpler than remote procedure calls between microservices. There's a whole new world of hurt in failure states, latency issues, and dependency update schedules that await those who venture into the lair of distribution."

> "That's the have-most-of-it-all we seek: All the power of individually tuned and distributed applications with the ease-of-use and understanding of a single, integrated system."

---

## 2. MVC in Rails (what the official sources say)

### 2.1 Getting Started guide

Source: https://guides.rubyonrails.org/getting_started.html (section 3.4, "Model-View-Controller Basics"; "The Rails philosophy" in section 2)

> "Rails code is organized using the Model-View-Controller (MVC) architecture. With MVC, we have three main concepts where the majority of our code lives:"
>
> - "Model - Manages the data in your application. Typically, your database tables."
> - "View - Handles rendering responses in different formats like HTML, JSON, XML, etc."
> - "Controller - Handles user interactions and the logic for each request."

The guide's two stated guiding principles:

> "The Rails philosophy includes two major guiding principles:"
>
> - "Don't Repeat Yourself: DRY is a principle of software development which states that 'Every piece of knowledge must have a single, unambiguous, authoritative representation within a system'. By not writing the same information over and over again, our code is more maintainable, more extensible, and less buggy."
> - "Convention Over Configuration: Rails has opinions about the best way to do many things in a web application, and defaults to this set of conventions, rather than require that you define them yourself through endless configuration files."

And the warning that precedes them:

> "If you persist in bringing old habits from other languages to your Rails development, and trying to use patterns you learned elsewhere, you may have a less happy experience."

### 2.2 Active Record Basics guide: where the model layer's responsibility sits

Source: https://guides.rubyonrails.org/active_record_basics.html (section 1, "What is Active Record?")

> "Active Record is part of the M in MVC - the model - which is the layer of the system responsible for representing data and business logic. Active Record helps you create and use Ruby objects whose attributes require persistent storage to a database."

> "What is the difference between Active Record and Active Model? It's possible to model data with Ruby objects that do not need to be backed by a database. Active Model is commonly used for that in Rails, making Active Record and Active Model both part of the M in MVC, as well as your own plain Ruby objects."

Note the explicit statement that the model layer is "responsible for representing data **and business logic**", and that the M includes Active Record, Active Model, and plain Ruby objects. This is the official basis for the "rich domain model" reading discussed in section 3.4.

### 2.3 The Doctrine's reading of the three layers

See section 1.3 above: views are a "big pot of functions" by design, Presenters are "the rare rather than common fit", and the model is "the prime bastion of object-oriented goodness".

---

## 3. Active Record

### 3.1 Why: the pattern definitions (Fowler) and Rails' adoption of them

**Fowler, Active Record catalog page**, https://martinfowler.com/eaaCatalog/activeRecord.html (dated 05 March 2003):

> "An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data."

> "An object carries both data and behavior. Much of this data is persistent and needs to be stored in a database. Active Record uses the most obvious approach, putting data access logic in the domain object. This way all people know how to read and write their data to and from the database."

**Fowler, Data Mapper catalog page**, https://martinfowler.com/eaaCatalog/dataMapper.html (dated 05 March 2003):

> "A layer of mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself."

> "Objects and relational databases have different mechanisms for structuring data. Many parts of an object, such as collections and inheritance, aren't present in relational databases. When you build an object model with a lot of business logic it's valuable to use these mechanisms to better organize the data and the behavior that goes with it. Doing so leads to variant schemas; that is, the object schema and the relational schema don't match up."

> "You still need to transfer data between the two schemas, and this data transfer becomes a complexity in its own right. If the in-memory objects know about the relational database structure, changes in one tend to ripple to the other."

> "The Data Mapper is a layer of software that separates the in-memory objects from the database. Its responsibility is to transfer data between the two and also to isolate them from each other. With Data Mapper the in-memory objects needn't know even that there's a database present; they need no SQL interface code, and certainly no knowledge of the database schema. (The database schema is always ignorant of the objects that use it.) Since it's a form of Mapper, Data Mapper itself is even unknown to the domain layer."

Both catalog pages end with "for more details go to Chapter 10 of the online ebook at oreilly.com"; the catalog pages themselves do not carry the book's "When to Use It" subsection (see section 7). Fowler's "when to use it" guidance for Active Record is, however, quoted from the book by Manrubia (dev.37signals.com, "Active Record, nice and blended"):

> "Even Fowler mentions that the pattern is a good choice for domain logic that isn't too complex."

> "When presenting the original pattern, Fowler argues that: If your business logic is complex, you'll soon want to use your object's direct relationships, collections, inheritance, and so forth. These don't map easily onto Active Record, and adding them piecemeal gets very messy."

**Rails Guides naming Fowler**, https://guides.rubyonrails.org/active_record_basics.html (section 1.1, "The Active Record Pattern"):

> "The term 'Active Record' also refers to a software architecture pattern. Active Record in Rails is an implementation of that pattern. It's also a description of something called an Object Relational Mapping system."

> "The Active Record pattern is described by Martin Fowler in the book Patterns of Enterprise Application Architecture as 'an object that wraps a row in a database table, encapsulates the database access, and adds domain logic to that data.' Active Record objects carry both data and behavior. Active Record classes match very closely to the record structure of the underlying database. This way users can easily read from and write to the database, as you will see in the examples below."

Section 1.3, "Active Record as an ORM Framework":

> "Active Record gives us the ability to do the following using Ruby objects:"
> - "Represent models and their data."
> - "Represent associations between models."
> - "Represent inheritance hierarchies through related models."
> - "Validate models before they get persisted to the database."
> - "Perform database operations in an object-oriented fashion."

Section 2, "Convention over Configuration in Active Record":

> "When writing applications using other programming languages or frameworks, it may be necessary to write a lot of configuration code. This is particularly true for ORM frameworks in general. However, if you follow the conventions adopted by Rails, you'll write very little to no configuration code when creating Active Record models."

> "Rails adopts the idea that if you configure your applications in the same way most of the time, then that way should be the default. Explicit configuration should be needed only in those cases where you can't follow the convention."

**ActiveRecord::Base doc comment**, https://raw.githubusercontent.com/rails/rails/main/activerecord/lib/active_record/base.rb :

> "Active Record objects don't specify their attributes directly, but rather infer them from the table definition with which they're linked. Adding, removing, and changing attributes and their type is done directly in the database. Any change is instantly reflected in the Active Record objects. The mapping that binds a given Active Record class to a certain database table will happen automatically in most common cases, but can be overwritten for the uncommon ones."

**Why Rails chose it (first-party rationale):**

- Doctrine, "No one paradigm": "Yes, because it proved to be the practical way to skin a web-app cat that virtually always talks to a database of some sort to persist the state of the domain model." (https://rubyonrails.org/doctrine)
- Doctrine, "Convention over Configuration": "If we can depend on a Person class mapping to people table, we can use that same inflection to map an association declared as has_many :people to look for a Person class."
- Manrubia, "Active Record, nice and blended" (https://dev.37signals.com/active-record-nice-and-blended/), standfirst: "Active Record restates the traditional question of how to separate persistence from domain logic: what if you don't have to?"

> "Because it can't be fully abstracted, many patterns aim to isolate database access on its own layer to keep the domain model persistence-free. For example repositories, Data Mappers or DAOs (Data Access Objects). Rails, however, went with a different approach — Active Record — a pattern introduced by Martin Fowler in Patterns of EAA"

> "The distinctive trait of the Active Record pattern is combining domain logic and persistence in the same class, and that's how we use it here at 37signals. At first glance, this might not look like a good idea. Shouldn't we keep separated things, well, separated? Even Fowler mentions that the pattern is a good choice for domain logic that isn't too complex. It's our experience, however, that Rails' Active Record keeps code elegant and maintainable in large and complex codebases."

> "I believe Active Record — the Rails framework, not the pattern — works so well in practice because it reduces this impedance mismatch to a minimum. There are two main reasons: It looks and feels like Ruby, even when you need to go lower level to fine-tune things. It comes with fantastic and innovative answers for recurring needs when dealing with objects and relational persistence."

> "Rails' Active Record offers answers for those and more. To enum a few: associations, single table inheritance, serialized attributes or delegated types."

On why not a separate persistence layer (the Data Mapper trade):

> "In theory, a rigid separation of persistence and domain logic sounds like a good idea. However, in practice, it comes with two significant challenges."

> "First, whatever approach you use will imply adding and orchestrating additional data-access abstractions in multiples of the number of persisted models in your app. That translates into additional ceremony and complexity."

> "Second, building rich domain models becomes harder. If domain models are free of persistence concerns, how can they implement business logic that needs to access the database? In DDD, for example, the answer is adding additional domain-level elements such as repositories and aggregates. Now you have three elements to coordinate, with plain domain entities knowing nothing about persistence. How do they access each other? How do they collaborate? This is a tempting scenario to come up with a service to orchestrate everything. It is not surprising that, ironically, many implementations that aim to embrace the best design practices end up suffering the anemic domain model problem, where most entities are mere data holders without behavior."

> "Active Record, however, is designed on the premise that persistence and domain logic belongs together, and it comes with almost two decades of iteration around this idea."

> "In a database-powered application, domain logic is indissolubly linked to persistence. Is isolating persistence a goal or mitigation against not having the right ORM (Object Relational Mapping) technology in place? My point of view is that if the ORM perfectly blends with the host language, and if it comes with good answers for persisting your object models, and if it offers good encapsulation mechanisms, then the original question gets restated. Instead of 'how can I isolate persistence from domain logic', it becomes 'why would I'?"

> "I believe Active Record is so good that it eliminates the traditional reasons for a strict separation of persistence and domain logic. I consider that a feature to embrace proudly, not a choice to justify."

### 3.2 Trade-offs the sources themselves acknowledge

| Trade-off | Who says it | Verbatim |
|---|---|---|
| Convention can mislead you into templating everything | Doctrine, CoC | "the power of convention isn't without peril ... most applications worth building have some elements that are unique in some way. It may only be 5% or 1%, but it's there." |
| Mixing persistence and domain logic "offends some purists" | Doctrine, No one paradigm | "We're mixing the logic needed for interfacing with the database directly with the business domain and logic. Such conflation of boundaries!" |
| Multi-paradigm costs "conceptual overhead" | Doctrine, No one paradigm | "the cost of this polyamorous relationship with the many paradigms of programming is conceptual overhead." |
| You still have to learn SQL for hard queries | Doctrine, No one paradigm | "We don't try to shield you that much from having to learn, say, JavaScript for the view or SQL for the occasional complicated query." |
| Concerns can become junk drawers | Doctrine, Sharp knives | "The charge is that concerns provide programmers prone to bloat their objects with a whole new set of drawers to stuff their clutter in. And that's true. Concerns can indeed be used like that." |
| Concerns proliferate methods on objects | DHH, "Put chubby models on a diet with concerns" (2012) | "It's true that this will lead to a proliferation of methods on some objects, but that has never bothered me." |
| Fowler's own limit on the pattern | Fowler via Manrubia | "If your business logic is complex, you'll soon want to use your object's direct relationships, collections, inheritance, and so forth. These don't map easily onto Active Record, and adding them piecemeal gets very messy." |
| Active Record needs schema control | Manrubia, Nice and blended | "The caveat here is that Active Record requires you to control the database schema to leverage what it offers fully." |
| Tool does not replace design skill | Manrubia, Nice and blended | "Active Record is a tool, and of course, you can use it to write messy and unmaintainable code. In particular, like it happens with concerns, it doesn't replace the need to design your system properly and it won't help you to do that if you don't know how." |
| Fat-model risk from the "rich model" approach | Manrubia, Vanilla Rails | "Because we like to access business logic via domain models, some core domain entities end up offering much functionality. How do we avoid the issues related to the dreaded fat model problem?" |
| Unknown at very large scale | Manrubia, Vanilla Rails | "I don't know if our approach would work at Shopify scale, but I am sure it would for most businesses using Rails out there." |
| Callbacks hide business rules | Callbacks guide | "These methods should be used with caution because there may be important business rules and application logic in callbacks that you do not want to bypass. Bypassing them without understanding the potential implications may lead to invalid data." |
| Callback side effects | Callbacks guide | "Refrain from using methods like update, save, or any other methods that cause side effects on the object within your callback methods." and "Using the Active Record Suppressor can introduce complexity and unexpected behavior. Suppressing saving can obscure the intended flow of your application" |
| STI table bloat | Associations guide, 5.9 | "A disadvantage of this approach is that it can result in table bloat, as the table will include attributes specific to each subclass, even if they aren't used by others. This can be solved by using Delegated Types." |
| Lazy loading / N+1 | Query Interface guide, 16.1 | "Retrieving a list of records N (where N is a number greater that 1) in a single query can sometimes trigger N extra queries; one for each record." |
| Delegated types make cross-table queries harder | DelegatedType doc comment | "A consequence of delegated types is that querying attributes spread across multiple classes becomes slightly more tricky, but not impossible." |

Sources: https://rubyonrails.org/doctrine ; https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns ; https://dev.37signals.com/active-record-nice-and-blended/ ; https://dev.37signals.com/vanilla-rails-is-plenty/ ; https://guides.rubyonrails.org/active_record_callbacks.html ; https://guides.rubyonrails.org/association_basics.html ; https://guides.rubyonrails.org/active_record_querying.html ; https://raw.githubusercontent.com/rails/rails/main/activerecord/lib/active_record/delegated_type.rb

### 3.3 Modern mitigations ("sharp knives"), with the framework's own doc comments

#### 3.3.1 `ActiveSupport::Concern`

Source: https://raw.githubusercontent.com/rails/rails/main/activesupport/lib/active_support/concern.rb

The doc comment motivates the module by contrast with the raw Ruby idiom (`def self.included(base); base.extend ClassMethods; base.class_eval do ... end; end`) and then:

> "By using `ActiveSupport::Concern` the above module could instead be written as:" (followed by the `extend ActiveSupport::Concern` / `included do` / `class_methods do` form)

> "Moreover, it gracefully handles module dependencies."

> "But why should `Host` care about `Bar`'s dependencies, namely `Foo`? We could try to hide these from `Host` directly including `Foo` in `Bar`:"

> "Unfortunately this won't work, since when `Foo` is included, its `base` is the `Bar` module, not the `Host` class. With `ActiveSupport::Concern`, module dependencies are properly resolved:"

> "Just like `include`, concerns also support `prepend` with a corresponding `prepended do` callback. `module ClassMethods` or `class_methods do` are prepended as well."

Original intent, DHH, "Put chubby models on a diet with concerns" (Dec 18 2012), https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns :

> "Different models in your Rails application will often share a set of cross-cutting concerns. In Basecamp, we have almost forty such concerns with names like Trashable, Searchable, Visible, Movable, Taggable."

> "These concerns encapsulate both data access and domain logic about a certain slice of responsibility."

> "Concerns are also a helpful way of extracting a slice of model that doesn't seem part of its essence (what is and isn't in the essence of a model is a fuzzy line and a longer discussion) without going full-bore Single Responsibility Principle and running the risk of ballooning your object inventory."

> "But I find that concerns are often just the right amount of abstraction and that they often result in a friendlier API. I far prefer current_account.posts.visible_to(current_user) to involving a third query object. And of course things like Taggable that needs to add associations to the target object are hard to do in any other way."

> "In Rails 4, we're going to invite programmers to use concerns with the default app/models/concerns and app/controllers/concerns directories that are automatically part of the load path. Together with the ActiveSupport::Concern wrapper, it's just enough support to make this light-weight factoring mechanism shine."

#### 3.3.2 `normalizes`

Current location on `main`: `ActiveModel::Attributes::Normalization`, https://raw.githubusercontent.com/rails/rails/main/activemodel/lib/active_model/attributes/normalization.rb (the Active Record file `activerecord/lib/active_record/normalization.rb` returns 404 on `main`; it exists on `8-0-stable`: https://raw.githubusercontent.com/rails/rails/8-0-stable/activerecord/lib/active_record/normalization.rb).

`main` doc comment on `normalizes`:

> "Declares a normalization for one or more attributes. The normalization is applied when the attribute is assigned or validated."

> "Because the normalization may be applied multiple times, it should be *idempotent*. In other words, applying the normalization more than once should have the same result as applying it only once."

> "By default, the normalization will not be applied to `nil` values. This behavior can be changed with the `:apply_to_nil` option."

> "#### Behavior with Active Record — To prevent confusion, normalization will not be applied when the attribute is fetched from the database. This means that if a record was persisted before the normalization was declared, the record's attribute will not be normalized until either it is assigned a new value, or it is explicitly migrated via Normalization#normalize_attribute."

8-0-stable (Active Record) doc comment on `normalizes`:

> "Declares a normalization for one or more attributes. The normalization is applied when the attribute is assigned or updated, and the normalized value will be persisted to the database. The normalization is also applied to the corresponding keyword argument of query methods. This allows a record to be created and later queried using unnormalized values."

The canonical example in both files is the one the authentication generator emits: `normalizes :email, with: -> email { email.strip.downcase }`.

#### 3.3.3 `delegated_type` (why over abstract classes / STI / bare polymorphism)

Source: https://raw.githubusercontent.com/rails/rails/main/activerecord/lib/active_record/delegated_type.rb (module doc comment)

> "Class hierarchies can map to relational database tables in many ways. Active Record, for example, offers purely abstract classes, where the superclass doesn't persist any attributes, and single-table inheritance, where all attributes from all levels of the hierarchy are represented in a single table. Both have their places, but neither are without their drawbacks."

> "The problem with purely abstract classes is that all concrete subclasses must persist all the shared attributes themselves in their own tables (also known as class-table inheritance). This makes it hard to do queries across the hierarchy."

> "How do you show a feed that has both `Message` and `Comment` records, which can be easily paginated? Well, you can't! Messages are backed by a messages table and comments by a comments table. You can't pull from both tables at once and use a consistent OFFSET/LIMIT scheme."

> "You can get around the pagination problem by using single-table inheritance, but now you're forced into a single mega table with all the attributes from all subclasses. No matter how divergent. If a Message has a subject, but the comment does not, well, now the comment does anyway! So STI works best when there's little divergence between the subclasses and their attributes."

> "But there's a third way: Delegated types. With this approach, the 'superclass' is a concrete class that is represented by its own table, where all the superclass attributes that are shared amongst all the 'subclasses' are stored. And then each of the subclasses have their own individual tables for additional attributes that are particular to their implementation. This is similar to what's called multi-table inheritance in Django, but instead of actual inheritance, this approach uses delegation to form the hierarchy and share responsibilities."

> "As you can see, neither `Message` nor `Comment` are meant to stand alone. Crucial metadata for both classes resides in the `Entry` 'superclass'. But the `Entry` absolutely can stand alone in terms of querying capacity in particular."

> "The entry 'superclass' also serves as a perfect place to put all that shared logic that applies to both messages and comments, and which acts primarily on the shared attributes."

> "If you need more complicated composition, or you need to perform dependent validation, you should build a factory method or class to take care of the complicated needs."

> "The delegated type shouldn't just answer the question of what the underlying class is called. In fact, that's an anti-pattern most of the time. The reason you're building this hierarchy is to take advantage of polymorphism."

Guide confirmation (https://guides.rubyonrails.org/association_basics.html, section 6):

> "Delegated types solves the Single Table Inheritance (STI) problem of table bloat via delegated_type. This approach allows us to store shared attributes in a superclass table and have separate tables for subclass-specific attributes."

Manrubia notes the origin: "A core domain entity in Basecamp is a Recording. Most elements a user manages in Basecamp are recordings, the original use case that motivated Rails' delegated types." (https://dev.37signals.com/vanilla-rails-is-plenty/)

#### 3.3.4 `strict_loading` / `strict_loading_by_default`

Source (instance API): https://raw.githubusercontent.com/rails/rails/main/activerecord/lib/active_record/core.rb

> "Sets the record to strict_loading mode. This will raise an error if the record tries to lazily load an association."

> "NOTE: Strict loading is disabled during validation in order to let the record validate its association."

> "`:mode` - Symbol specifying strict loading mode. Defaults to :all. Using :n_plus_one_only mode will only raise an error if an association that will lead to an n plus one query is lazily loaded."

The class attributes are declared in the same file without prose doc comments: `class_attribute :strict_loading_by_default, instance_accessor: false, default: false` and `class_attribute :strict_loading_mode, instance_accessor: false, default: :all`. The behaviour is documented in the guides:

Query Interface guide, https://guides.rubyonrails.org/active_record_querying.html (16.5, 16.6):

> "Eager loading can prevent N + 1 queries but you might still be lazy loading some associations. To make sure no associations are lazy loaded you can enable strict_loading."

> "By enabling strict loading mode on a relation, an ActiveRecord::StrictLoadingViolationError will be raised if the record tries to lazily load any association"

> "To enable strict loading for all relations, change config.active_record.strict_loading_by_default to true" ... "To send violations to the logger instead, change config.active_record.action_on_strict_loading_violation to :log"

Configuring guide, https://guides.rubyonrails.org/configuring.html (3.8.31–3.8.33):

> "config.active_record.action_on_strict_loading_violation — Enables raising or logging an exception if strict_loading is set on an association. The default value is :raise in all environments. It can be changed to :log to send violations to the logger instead of raising."

> "config.active_record.strict_loading_by_default — Is a boolean value that either enables or disables strict_loading mode by default. Defaults to false."

> "config.active_record.strict_loading_mode — Sets the mode in which strict loading is reported. Defaults to :all. It can be changed to :n_plus_one_only to only report when loading associations that will lead to an N + 1 query."

Associations guide, option `:strict_loading` (8.1.9): "Enforces strict loading every time an associated record is loaded through this association."

#### 3.3.5 `ActiveModel::API` / `ActiveModel::Model` (POROs in forms and as domain models)

Source: https://raw.githubusercontent.com/rails/rails/main/activemodel/lib/active_model/api.rb

> "Includes the required interface for an object to interact with Action Pack and Action View, using different Active Model modules. It includes model name introspections, conversions, translations, and validations. Besides that, it allows you to initialize the object with a hash of attributes, pretty much like Active Record does."

> "Note that, by default, `ActiveModel::API` implements #persisted? to return `false`, which is the most common case. You may want to override it in your class to simulate a different scenario"

Source: https://raw.githubusercontent.com/rails/rails/main/activemodel/lib/active_model/model.rb

> "Allows implementing models similar to ActiveRecord::Base. Includes ActiveModel::API for the required interface for an object to interact with Action Pack and Action View, but can be extended with other functionalities."

`ActiveModel::Model` is `include ActiveModel::API` plus `include ActiveModel::Access` (`slice`, `values_at`). Manrubia's sign-up example uses exactly this: `class Project::InvitationToken::Signup; include ActiveModel::Model; include ActiveModel::Validations::Callbacks ...` with the comment "So instead of having a SigningUpService in charge of the 'signing up' domain operation, we have a Signup class that lets you validate and create an identity in the app."

#### 3.3.6 Deprecated associations (Rails 8.1)

Source: https://guides.rubyonrails.org/8_1_release_notes.html (2.6) and https://guides.rubyonrails.org/association_basics.html (8.1.12)

> "Active Record associations can now be marked as being deprecated: `has_many :posts, deprecated: true` ... With that, usage of the posts association will be reported."

> "Three reporting modes are supported (:warn, :raise, and :notify), and backtraces can be enabled or disabled, though you always get the location of the reported usage regardless. Defaults are :warn mode and disabled backtraces."

#### 3.3.7 Callback hygiene (guide guidance)

Source: https://guides.rubyonrails.org/active_record_callbacks.html

> "Callbacks allow you to trigger logic before or after a change to an object's state. They are methods that get called at certain moments of an object's life cycle."

> "It is considered good practice to declare callback methods as private. If left public, they can be called from outside of the model and violate the principle of object encapsulation."

> "Refrain from using methods like update, save, or any other methods that cause side effects on the object within your callback methods. For instance, avoid calling update(attribute: 'value') inside a callback. This practice can modify the model's state and potentially lead to unforeseen side effects during commit. Instead, you can assign values directly (e.g., self.attribute = 'value') in before_create, before_update, or earlier callbacks for a safer approach."

### 3.4 Where business logic lives: the 37signals position (Manrubia) and DHH's framing

Source: Jorge Manrubia, "Vanilla Rails is plenty", Nov 8 2022, https://dev.37signals.com/vanilla-rails-is-plenty/

The critique being answered:

> "I have often heard this: vanilla Rails can only get you so far. At some point, apps become unmaintainable, and you need a different approach that brings the missing pieces, because Rails encourages a poor separation of concerns at the architectural level."

> "But Rails only offers controllers and models: models include persistence via Active Record, and Rails encourages direct access to them from controllers. Critics argue that the application, domain, and infrastructure layers inevitably merge into a single mess of fat models."

The position:

> "We don't separate application-level and domain-level artifacts. Instead, we have a set of domain models (both Active Records and POROs) exposing public interfaces to be invoked from the system boundaries, typically controllers or jobs. We don't separate that API from the domain model, architecturally speaking."

> "In other words, we don't default to create services, actions, commands, or interactors to implement controller actions."

> "We are fine with plain CRUD accesses from controllers for simple scenarios."

> "Most of our controllers use this approach of accessing models directly: a model exposes a method, and the controller invokes it."

> "As opposed to anemic domain models, our approach encourages building rich domain models. We think of domain models as our application API and, as a guiding design principle, we want that one to feel as natural as possible."

> "Because we like to access business logic via domain models, some core domain entities end up offering much functionality. How do we avoid the issues related to the dreaded fat model problem? With two tactics: Using concerns to organize model's code. Delegating functionality to additional systems of objects (AKA using plain object-oriented programming)."

> "This example is not as simple as the incineration one, but the principle is the same: rich internal object models hidden behind high-level APIs on domain models. This doesn't mean we always create additional classes to implement concerns, far from it, but we do when complexity justifies it."

> "In our example, there are no fat models in charge of doing too many things. Recording::Incineration or Recording::Copier are cohesive classes that do one thing. Recording::Copyable adds a high-level #copy_to method to Recording's public API and keeps the related code and data definitions separated from other Recording responsibilities."

On services:

> "We don't use services as first-class architectural artifacts in the DDD sense (stateless, named after a verb), but we have many classes that exist to encapsulate operations. We don't call those services and they don't receive special treatment. We usually prefer to present them as domain models that expose the needed functionality instead of using a mere procedural syntax to invoke the operation."

> "Also, we don't make a big deal of distinguishing whether a domain model is persisted or not (Active record or PORO). From the business logic consumer's point of view, that's irrelevant, so we don't capture the distinction between domain entities and value objects in code. They are both domain models to us. You can find many POROs in our app/models folders."

On the risk of the alternative:

> "I believe such nuance-free recommendations favor the appearance of either: Tons of boilerplate code because many of these application-level elements simply delegate the operation to some domain entity ... An anemic domain model, where the application-level elements are the ones implementing the business rules, and domain models become empty shells carrying data around."

> "Our approach reflects one of the Rails' doctrine pillars: No one paradigm."

Scale claim:

> "As a recent example, we just launched Basecamp 4 built on top of Basecamp 3, the codebase of which is almost 9 years old, includes 400 controllers and 500 models, and serves millions of users every day."

Callbacks in practice (from "Active Record, nice and blended"): the `Filing` example uses `after_create_commit :process_later, unless: :completed?` to enqueue a job; encapsulation is via concerns and private methods:

> "Because it blends so well with Ruby, Active Record plays great with using regular Ruby goodies to hide details away. This allows you to write code that looks natural and encapsulates persistence concerns without paying the ceremony tax of a separate data-access layer."

> "For querying, we use scopes extensively. Combining those with associations and other scopes lets you express complex queries with natural-looking code."

**DHH, "The Majestic Monolith"** (Feb 29 2016), https://m.signalvnoise.com/the-majestic-monolith/ :

> "The problem with prematurely turning your application into a range of services is chiefly that it violates the #1 rule of distribute computing: Don't distribute your computing! At least if you can in any way avoid it."

> "So what is a majestic monolith exactly? It's an integrated system that collapses as many unnecessary conceptual models as possible. Eliminates as much needless abstraction as you can swing a hammer at. It's a big fat no to distributing your system lest it truly prevents you from doing what really needs to be done."

> "One of the benefits to the majestic monolith is that it basically presumes that the people who work on it also understand it."

> "TL;DR: Run a small team, not a tech behemoth? Embrace the monolith and make it majestic. You Deserve It!"

DHH in the comments (May 20, 2019): "I don't see any fundamental improvement that alters the equation. Integrate your systems until it's impossible for one person to hold it all in their head. We still haven't gotten there with Basecamp or any of our apps."

**DHH, Rails 8.0 announcement** (Nov 7 2024), https://rubyonrails.org/2024/11/7/rails-8-no-paas-required, the only first-party text found this session that uses the "compressing complexity" phrasing:

> "Whether you're into #NOBUILD or #NOPAAS or simply attracted to the mission of compressing complexity in general, you'll be right at home with a community of passionate builders who value beautiful code as much as they do productivity."

---

## 4. Authentication and authorization intent

### 4.1 The intent, in DHH's words (issue #50446)

Source: https://github.com/rails/rails/issues/50446 ("Add basic authentication generator", opened by dhh, December 26, 2023; closed July 16, 2024 with the comment "Closed via #52328."). Note: #50446 is the **issue**, not the PR. Text recovered via WebFetch, which limits verbatim quotes to 125-character fragments; fragments are reassembled below in order, with one fragment paraphrased where marked.

> "Rails now include all the key building blocks needed to do basic authentication," "but many new developers are still uncertain of how to put them together," "so they end up leaning on all-in-one gems that hide the mechanics."

> "While these gems are great, and many people enjoy using them, they should not be seen as a necessity."

> "We can teach Rails developers how to use the basic blocks by adding a basic authentication generator" [remainder paraphrased by the fetcher: the generator would act as a scaffold, but for authentication rather than resources].

> "Note: This ticket serves as a placeholder for the intention to do this work." "Not as an open invitation to feature requests."

> "Before we engage in broad discussion of what should or shouldn't be in such a generator," "the core team will propose a solution first," "so there's something tangible to discuss."

### 4.2 The PR that shipped it (#52328)

Source: https://github.com/rails/rails/pull/52328, title "Add basic sessions generator", author dhh, merged July 16, 2024. Opening post (same fragment caveat):

> "Adds a basic sessions generator to get people started with their own authentication system."

> "This is not intended to be an all-singing, all-dancing answer to every possible authentication concern."

> "It's merely intended to illuminate the basic path," "and reveal that rolling your own authentication system is not some exotic adventure."

> "So do not expect magic links or passkeys or 2FA." "That's not going to happen with this generator."

> "Outstanding work:" "Add tests"

Generator USAGE on `main` (https://raw.githubusercontent.com/rails/rails/main/railties/lib/rails/generators/rails/authentication/USAGE): "Generates a basic authentication system with users, sessions, and password reset."

### 4.3 What the generator emits (verbatim templates on `main`)

`app/controllers/concerns/authentication.rb`, https://raw.githubusercontent.com/rails/rails/main/railties/lib/rails/generators/rails/authentication/templates/app/controllers/concerns/authentication.rb.tt

```ruby
module Authentication
  extend ActiveSupport::Concern

  included do
    before_action :require_authentication
    helper_method :authenticated?
  end

  class_methods do
    def allow_unauthenticated_access(**options)
      skip_before_action :require_authentication, **options
    end
  end

  private
    def authenticated?
      resume_session
    end

    def require_authentication
      resume_session || request_authentication
    end

    def resume_session
      Current.session ||= find_session_by_cookie
    end

    def find_session_by_cookie
      Session.find_by(id: cookies.signed[:session_id]) if cookies.signed[:session_id]
    end

    def request_authentication
      session[:return_to_after_authenticating] = request.url
      redirect_to new_session_path
    end

    def after_authentication_url
      session.delete(:return_to_after_authenticating) || root_url
    end

    def start_new_session_for(user)
      user.sessions.create!(user_agent: request.user_agent, ip_address: request.remote_ip).tap do |session|
        Current.session = session
        cookies.signed.permanent[:session_id] = { value: session.id, httponly: true, same_site: :lax }
      end
    end

    def terminate_session
      Current.session.destroy
      cookies.delete(:session_id)
    end
end
```

`app/models/user.rb.tt`: `has_secure_password` / `has_many :sessions, dependent: :destroy` / `normalizes :email_address, with: ->(e) { e.strip.downcase }`

`app/models/session.rb.tt`: `belongs_to :user`

`app/models/current.rb.tt`: `class Current < ActiveSupport::CurrentAttributes; attribute :session; delegate :user, to: :session, allow_nil: true; end`

`app/controllers/sessions_controller.rb.tt`: `allow_unauthenticated_access only: %i[ new create ]`, `rate_limit to: 10, within: 3.minutes, only: :create, with: -> { redirect_to new_session_path, alert: "Try again later." }`, and `if user = User.authenticate_by(params.permit(:email_address, :password))`.

Design observations grounded in the above: authentication is a controller **concern** (section 3.3.1), the current user is reached via `Current.session.user` (an `ActiveSupport::CurrentAttributes`), sessions are first-class Active Record rows (not just a user id in the cookie), and the model uses `normalizes` (section 3.3.2). Password reset and the mailer are also generated (see the Security guide file list).

### 4.4 Rails 8 announcement on authentication

Source: https://rubyonrails.org/2024/11/7/rails-8-no-paas-required

> "Finally, making it easier to go to production also means we ought to make it easy to be secure. Rails has been assembling high-level abstractions for the key components of an excellent authentication system for a long time to bring that ease. We've had has_secure_password since Rails 5, but also recently introduced generates_token_for :password_reset along with authenticate_by in Rails 7.1. Now, with Rails 8, we're putting all the pieces together in a complete authentication system generator, which creates an excellent starting point for a session-based, password-resettable, metadata-tracking authentication system."

> "Just run bin/rails generate authentication and you'll get basic models for Session and User, together with a PasswordsMailer, SessionsController, and an Authentication concern. All you have to bring yourself is a user sign-up flow (since those are usually bespoke to each application). No need to fear rolling your own authentication setup with these basics provided (or, heaven forbid, paying a vendor for it!)."

### 4.5 Security guide: authentication section and authorization via scoping

Source: https://guides.rubyonrails.org/security.html

Section 2, "Authentication":

> "Starting with version 8.0, Rails comes with a default authentication generator, which provides a solid starting point for securing your application by only allowing access to verified users."

> "After running the Authentication generator, you do need to implement your own sign up flow and add the necessary views, routes, and controller actions. There is no code generated that creates new user records and allows users to 'sign up' in the first place."

> "has_secure_password adds the following validations automatically: Password must be present on creation; Password length should be less than or equal to 72 bytes; Confirmation of password (using a XXX_confirmation attribute). However it doesn't validate the minimum length or the complexity of the password, you need to define validation for those yourself."

> "The core functionality around session management is implemented in the Authentication controller concern, which is included by the ApplicationController in your application."

> "You can find all of the details for the Authentication generator in the Rails source code. You are encouraged to explore the implementation details and not treat authentication as a black box."

Section 6.6, "Privilege Escalation", the guide's authorization pattern (scoping through the current user):

> "Changing a single parameter may give the user unauthorized access. Remember that every parameter may be changed, no matter how much you hide or obfuscate it."

> "The most common parameter that a user might tamper with, is the id parameter, as in http://www.domain.com/project/1, whereas 1 is the id. It will be available in params in the controller. There, you will most likely do something like this: `@project = Project.find(params[:id])`"

> "This is alright for some web applications, but certainly not if the user is not authorized to view all projects. If the user changes the id to 42, and they are not allowed to see that information, they will have access to it anyway. Instead, query the user's access rights, too: `@project = @current_user.projects.find(params[:id])`"

> "As a rule of thumb, no user input data is secure, until proven otherwise, and every parameter from the user is potentially manipulated."

Section 7.2.2, "Bypassing Authorization", shows the injection case `User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")` and notes "This will simply find the first record in the database and grant access to this user."

The same scoping idiom appears in DHH's 2012 concerns post (`current_account.posts.visible_to(current_user)`) and in Manrubia's HEY example (`@collection.topics.active.accessible_to(Acting.contact)`), i.e. authorization is expressed as association/scope chains off the current actor rather than as a separate authorization layer. The Security guide does not name an authorization library.

---

## 5. Rails 8.0 and 8.1: features and deprecations

### 5.1 Rails 8.0 (released November 7, 2024)

Source: https://guides.rubyonrails.org/8_0_release_notes.html ; https://rubyonrails.org/2024/11/7/rails-8-no-paas-required

| Feature | One line (from the release notes) |
|---|---|
| Kamal 2 | "Rails now comes preconfigured with Kamal 2 for deploying your application. Kamal takes a fresh Linux box and turns it into an application or accessory server with just a single 'kamal setup' command." Includes Kamal Proxy "to replace the generic Traefik option". |
| Thruster | "The Dockerfile has been upgraded to include a new proxy called Thruster, which sits in front of the Puma web server to provide X-Sendfile acceleration, asset caching, and asset compression." |
| Solid Cable | "Solid Cable replaces Redis to act as the pubsub server to relay WebSocket messages ... retains the messages sent in the database for a day by default." |
| Solid Cache | "Solid Cache replaces either Redis or Memcached for storing HTML fragment caches in particular." |
| Solid Queue | "Solid Queue replaces the need for Redis, also a separate job-running framework, like Resque, Delayed Job, or Sidekiq." Built on `FOR UPDATE SKIP LOCKED`; "It also works with SQLite." |
| Propshaft | "Propshaft is now the default asset pipeline, replacing the old Sprockets system." |
| Authentication generator | "Authentication system generator, creates a starting point for a session-based, password-resettable, metadata-tracking authentication system." |
| `params.expect` | "Introduce safer, more explicit params handling method with params#expect such that params.expect(table: [ :attr ]) replaces params.require(:table).permit(:attr)." |
| Regexp timeout | "Set Regexp.timeout to 1s by default to improve security over Regexp Denial-of-Service attacks." |
| db:migrate on fresh DB | "Running db:migrate on a fresh database now loads the schema before running migrations." |

Notable removals and deprecations in 8.0:

| Framework | Removals | Deprecations |
|---|---|---|
| Railties | `config.read_encrypted_secrets`; `rails/console/app`; `rails/console/helpers`; extending console via `Rails::ConsoleMethods` | requiring `"rails/console/methods"`; modifying `STATS_DIRECTORIES` (use `Rails::CodeStatistics.register_directory`); `bin/rake stats` (use `bin/rails stats`) |
| Action Pack | `config.action_controller.allow_deprecated_parameters_hash_equality` | "Deprecate drawing routes with multiple paths to make routing faster." |
| Action View | passing `nil` to `model:` of `form_with`; passing content to void tag elements |, |
| Active Record | `commit_transaction_on_non_local_return`; `allow_deprecated_singular_associations_name`; unregistered adapters; enum with keyword args; `warn_on_records_fetched_greater_than`; `sqlite3_deprecated_warning`; `ConnectionPool#connection`; db name to `cache_dump_filename`; `ENV["SCHEMA_CACHE"]` | SQLite3Adapter `retries` option (use `timeout`) |
| Active Storage |, | Azure backend |
| Active Support | `ActiveSupport::ProxyObject`; `attr_internal_naming_format` with `@` prefix; array of strings to `Deprecation#warn` | `Benchmark.ms`; addition/`since` between `Time` and `ActiveSupport::TimeWithZone` |
| Active Job | `config.active_job.use_big_decimal_serializer` | `enqueue_after_transaction_commit`; internal SuckerPunch adapter |

### 5.2 Rails 8.1 (released October 22, 2025)

Source: https://guides.rubyonrails.org/8_1_release_notes.html ; https://rubyonrails.org/2025/10/22/rails-8-1 ("Rails 8.1: Job continuations, structured events, local CI", posted by rafaelfranca: "Rails 8.1 represents the work of over 500 contributors across 2500 commits since our last major release.")

| Feature | One line (from the release notes) |
|---|---|
| Active Job Continuations | "Long-running jobs can now be broken into discrete steps that allow execution to continue from the last completed step rather than the beginning after a restart." (`include ActiveJob::Continuable`, `step :name`) |
| Structured Event Reporting | "The new Event Reporter provides a unified interface for producing structured events in Rails applications: `Rails.event.notify(...)`", with tags, context, and subscriber `#emit`. |
| Local CI | "Rails has therefore added a default CI declaration DSL, which is defined in config/ci.rb and run by bin/ci." Optional `gh signoff` integration. |
| Markdown Rendering | "making it easier to respond to markdown requests and render them directly" (`format.md { render markdown: @page }`). |
| Command-line Credentials Fetching | "Kamal can now easily grab its secrets from the encrypted Rails credentials store for deploys" (`rails credentials:fetch kamal.registry_password`). |
| Deprecated Associations | "Active Record associations can now be marked as being deprecated" (`has_many :posts, deprecated: true`); modes `:warn`, `:raise`, `:notify`. |
| Registry-Free Kamal Deployments | "Kamal no longer needs a remote registry, like Docker Hub or GHCR, to do basic deploys. By default, Kamal 2.8 will now use a local registry for simple deploys." |
| Verbose redirects | "Redirects are now verbose in development for new Rails apps." |
| schema.rb ordering | "The table columns inside schema.rb are now sorted alphabetically." |

Notable removals and deprecations in 8.1:

| Framework | Removals | Deprecations |
|---|---|---|
| Railties | `rails/console/methods.rb`; `bin/rake stats`; `STATS_DIRECTORIES` |, |
| Action Pack | skipping leading brackets in param names; semicolons as query-string separator; a route to multiple paths | `config.action_dispatch.ignore_leading_brackets` |
| Active Record | SQLite3 `:retries`; MySQL `:unsigned_float` / `:unsigned_decimal` | order-dependent finders (e.g. `#first`) without an order; `ActiveRecord::Base.signed_id_verifier_secret` (use `Rails.application.message_verifiers` / `Model.signed_id_verifier`); `insert_all`/`upsert_all` with unpersisted records in associations; `WITH`, `WITH RECURSIVE`, `DISTINCT` with `update_all` |
| Active Storage | `:azure` storage service |, |
| Active Support | passing `Time` to `Time#since`; `Benchmark.ms`; `Time + TimeWithZone`; `to_time` preserving system local time | `config.active_support.to_time_preserves_timezone`; `String#mb_chars` and `ActiveSupport::Multibyte::Chars`; `ActiveSupport::Configurable` |
| Active Job | `enqueue_after_transaction_commit` set to `:never`/`:always`/`:default`; `config.active_job.enqueue_after_transaction_commit`; internal SuckerPunch adapter | custom serializers must have public `#klass`; built-in sidekiq adapter (now in the sidekiq gem) |

---

## 6. Core team roster

Source: https://rubyonrails.org/community (fetched 2026-10-29). The page lists first names only, with country and "Since" year; the linked profile URL is given here as the disambiguator the page itself provides.

> "The direction of the framework is stewarded by the Rails Core. This group of long-term contributors manages releases, evaluates pull-requests, handles conduct complaints, and does a lot of the groundwork on major new features."

**Core**

| Name (as listed) | Country | Since | Linked profile |
|---|---|---|---|
| David | USA | 2003 | https://dhh.dk |
| Jeremy | USA | 2005 | https://twitter.com/bitsweat |
| Aaron | USA | 2011 | https://twitter.com/tenderlove |
| Xavier | Catalonia | 2011 | https://hashref.com |
| Rafael | Canada | 2012 | https://twitter.com/rafaelfranca |
| Guillermo | Canada | 2012 | https://twitter.com/guilleiguaran |
| Carlos | Brazil | 2012 | https://twitter.com/cantoniodasilva |
| Matthew | Australia | 2015 | https://twitter.com/_matthewd |
| Eileen | USA | 2017 | https://twitter.com/eileencodes |
| Ryuta | Japan | 2022 | https://blog.kamipo.net |
| Jean | France | 2022 | https://github.com/byroot |
| John | Canada | 2022 | https://github.com/jhawthorn |
| Mike | USA | 2026 | https://github.com/flavorjones |

> "The Committer team assists with processing pull requests and making changes to the framework, but does not have the keys to make final releases or set policy. All members of the Core team came up through working on this team:"

**Committers**

| Name (as listed) | Country | Linked profile |
|---|---|---|
| Akira | Japan | https://twitter.com/a_matsuda |
| Eugene | Ireland | https://twitter.com/eugeneius |
| Dima | Ukraine | https://github.com/fatkodima |
| Gannon | Canada | https://github.com/gmcgibbon |
| Hartley | USA | https://github.com/skipkayhil |
| Jonathan | USA | https://jonathan.hefner.pro |
| Yasuo | Japan | https://github.com/yahonda |

The page also lists an Issues team (Adam Hess, Adrianna Chang, Akhil G Krishnan, Alex Ghiculescu, Andrew Novoselac, Edouard Chin, Haroon Ahmed, Joshua Young, Jorge Manrubia, Luan Vieira, Matheus Richard, Nikita Vasilevsky, Paul Reece, Petrik de Heus, Prem Sichanugrist, Robin Dupret, Sean Doyle, Vipul A M, Zak Scott) and Alumni (George Claghorn, Santiago Pastorino, Yves Senn, Godfrey Chan, Michael Koziarski, José Valim, Yehuda Katz, Jon Leighton, Josh Peek, Carl Lerche, Pratik Naik, Jamis Buck, Marcel Molina, Nicholas Seckar, Sam Stephenson, Florian Weber, Scott Barron, Tobias Lütke, Rick Olson).

---

## 7. Claims I could not source (or could only partially source)

1. **DHH's "conceptual compression" essay.** The Signal v. Noise post "Conceptual compression means beginners don't need to know SQL, and that's a good thing" redirects from m.signalvnoise.com to signalvnoise.com and returns HTTP 404 there; the Wayback Machine was not reachable from this session. The phrase appears first-hand only as "the mission of compressing complexity in general" in the Rails 8.0 announcement (quoted in 3.4). All other hits were secondhand write-ups of the RailsConf 2018 keynote, which I did not use.
2. **DHH's "Rails is omakase" (2012) essay.** `david.heinemeierhansson.com` does not resolve from this session (host not in the allowed network list), so the original essay is not quoted; the Doctrine's "The menu is omakase" section (quoted in 1.2) is the first-party source used instead.
3. **A DHH text using the exact phrase "rich domain model".** Not found on rubyonrails.org, world.hey.com, or signalvnoise.com this session. The phrase is sourced to Jorge Manrubia (37signals Director of Product Development) in "Vanilla Rails is plenty". DHH's nearest first-party wording is the Doctrine's "we generally treat the model in our MVC layer cake as the prime bastion of object-oriented goodness".
4. **world.hey.com/dhh posts.** My one guessed URL returned the HEY World marketing page, not a post; no hey.com post is quoted. The "majestic monolith" and "vanilla Rails" positions are therefore sourced from the Doctrine, m.signalvnoise.com, and dev.37signals.com.
5. **Fowler's "When to Use It" subsections.** The catalog pages at martinfowler.com carry only the pattern definition and the first paragraphs; the "when to use it" guidance lives in the book (Chapter 10), which I could not fetch. The two Fowler sentences on Active Record's limits are quoted as reproduced by Manrubia.
6. **PR #52328 / issue #50446 verbatim text.** `gh api` and direct curl to github.com/api.github.com are blocked for this session (repository not attached; attaching with API credentials was denied). WebFetch could read both pages but only returns verbatim fragments of up to 125 characters; the quotations in section 4 are those fragments reassembled in order, with one clause of the issue text paraphrased as marked. The task's reference to "PR rails/rails#50446" is a misnumbering: #50446 is the issue; the PR is #52328, titled "Add basic sessions generator".
7. **Doc comments for `strict_loading_by_default`.** `core.rb` declares the class attributes without prose; the prose is in the Configuring and Query Interface guides (quoted in 3.3.4).
8. **`ActiveRecord::Normalization` on `main`.** The file no longer exists at `activerecord/lib/active_record/normalization.rb` on `main` (404); the implementation moved to `ActiveModel::Attributes::Normalization`. Both the `main` Active Model comment and the `8-0-stable` Active Record comment are quoted.
9. **Full names of Core and Committers.** rubyonrails.org/community publishes first names only; I did not add surnames from memory. The linked profile URLs are reproduced as the page's own disambiguator.
