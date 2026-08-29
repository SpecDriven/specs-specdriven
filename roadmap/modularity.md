# Modularity

In order to dispatch coding sessions to different coding agents...

In order to have a template workflow with pluggable behavior...

SpecDriven needs a notion of a core with extension points.

Mechanisms:
 - Versioned interfaces
 - Migration from older versions to newer versions
 - Validation of plugins? (the null-object cycle?)

Example Extension Points: W
 - Strategy for naming worktrees, templated variables?

Inspirations:
 - Strategy Pattern
 - Late Binding
 - Developer Customization
   - Serving Suggestion based on the interface
   - Open to local experiments (vibe coded plug-ins)
 - Enterprise Customization
   - Validate integrations with enterprise apps / services / systems
   - Enable repair and
