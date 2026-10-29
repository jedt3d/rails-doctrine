---
status: accepted
---
# Ruby 3.4 is the ceiling, not Ruby 4.0

Projects pin Ruby 3.4.x even though Ruby 4.0 is the current stable release (4.0.6 is installable in the lab environment) and Rails 8.1 only requires Ruby 3.2. The owner wants a small, stable language surface while the doctrine is being built: Ruby 3.4 gives the `it` block parameter, Prism as the default parser and chilled-string warnings; Ruby 4.0 adds ZJIT, Ruby::Box and Set in core, none of which a rule depends on yet. The ceiling is revisited when the doctrine's Ruby idiom rules are stable and evaluated.
