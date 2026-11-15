# AGENTS.md

Version-controlled source of the agent instructions that apply across all of Martin's projects.

## Language

**Global file**:
The user-level agent instructions loaded in every project on the machine.
_Avoid_: User file, base file

**Project file**:
A project's own agent instructions, committed in that project and loaded alongside the global file.
_Avoid_: Local file

**Project template**:
The starting point for a project file, copied once into a new project and owned by that project afterwards.
_Avoid_: Base, starter, scaffold
