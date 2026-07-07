## fact: The Rule of Zero beats the Rule of Five
tags: raii, core

If your class needs a custom destructor, copy constructor, copy assignment, move constructor, or move assignment — it almost certainly needs all five (Rule of Five). But the better goal is the **Rule of Zero**: own resources only through members that already manage themselves (`std::string`, `std::vector`, `std::unique_ptr`), and write none of the five.

Special members you don't write can't have bugs.
