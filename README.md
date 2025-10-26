# devops_simulation
simulation of two modes of devops
Must run with any client specifications from any domain/domains

Legacy folder and it's contents are read only, except contents of subfolder legacy/documentation and contents of README.md files in folder legacy and all it's subfolders. It is source code that must be refactored into clear modular code with the same behavior (+ some improvements, but at least the same behavior) and will be executed in Visual Studio 2022.
Aim of this code is multiple runs in standard DEVOPS mode (backword compatibility = true) and enhanced devops (backword compatibility = false), produce metrics, which will be later used to compare standard devops and enhanced devops. Both modes act on 2 json files.
Explanation/context:
Generally, standard devops is process of generation of software engineering documentation, when macroiterations may be from one to many, miniiterations always = 1, issues are being detected after each artifact/table is being generated, but are being fed into creator prompt only on next macroiteration. Lessons are learned only in the end of macroiteration, and shown to corresponding artifacts creator prompt only in next macroiteration.
In contrast, in enhanced devops  when macroiterations may be from one to many, miniiterations may be from one to many, issues are being detected after each artifact/table is being generated, and are being fed into creator prompt in next miniiteration. Lessons are learned only in the end of each miniiteration and macroiteration, and shown to corresponding artifacts creator prompt in next miniiteration/macroiteration.
As issues are being provided to creater prompt earlier and lessons learned are generated and provided to creator prompt earlier, enhanced devops is expected to produce artifact of higher quality (which I think may be reflected in either higher tabular complexity of artifacts of last macroiteration of enhanced devops vs standard devops, or lower concentration of issues of last macroiteration). However, as for now reporting system is working, it seems that it is not correctly implemented - each macroiteration of enhanced devops may contain more than one miniiteration, while standard devops has one only, wich skewes results of assessment of both modes and makes them incomparable. So later some additional reporting mode/layer? must be implemented, comparing really only last set of mini of last macro of enhanced devops mode vs last macro of standard devops, making them comparable.
However, to keep process starigtforward, we must proceed with the project in nexxt steps:
1. Document fully current behavior
2. Plan refactoring
3. Refactor
4. Plan enhancements
5. Implement enhancements. 
