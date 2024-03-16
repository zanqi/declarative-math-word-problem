# Findings from some tests
OpenAI deprecated the code-davinci-002 model used in the original paper, so the current recommended replacement model gpt-3.5-turbo-instruct is used in my tests. This model is not able to answer the following math word problem:
```
Bob says to Alice: if you give me 3 apples and then take half of my apples away, then I will be left with 13 apples. How many apples do I have now?
```
It produces the following equations:
```
b = a + 3,  c = b / 2,  d = c + 13, d = ?
```
These equations are not solvable by sympy, so the final answer is `no solution`. The desired equations should have `a = ?` instead of `d = ?` at the end. Upon examining the prompt, it is found that the prompt is designed to solve math word problems that have the last variable as the answer. That's why it generates the last equation as `d = ?`.

This suggests it might be helpful to add an example of a problem where the last variable is not the answer, but the first variable is the answer. The following example is added to the prompt:
```
Q: Mary has some apples. If she is given 2 apples, and then half of her apples was taken away, she will be left with 10 apples. How many apples does she have?

Let a be the number of apples Mary started with [[var a]]. 
Let b be how many apples she had after being given 2 apples [[var b]]. We have [[eq b = a + 2]].
Let c be how many apples she had after half of her apples was taken away [[var c]]. We have [[eq c = b / 2]]. We also have [[eq c = 10]].
The answer is the value of a [[answer a]].
```
With this new prompt, the equations generated by the model are:
```
b = 3,  c = a + b / 2,  c = 13, a = ?
```
These equations have a solution, but the answer 11.5 is wrong. The problem is the model tried to use a single equation `c = a + b / 2` to describe the sentence `if you give me 3 apples and then take half of my apples away`. This could work if the model understand math precedence is use parenthesis `c = (a + b) / 2`, but that not fair to expect it from the model given our prompt does not demonstrate such math concept. 

Another way around the issue is it could have used two equations `b = a + 3` and `c = b / 2` to describe it. Our prompt uses a format of one sentence producing one equation, this suggests adding a comma in the problem to help the model `if you give me 3 apples, and then take half of my apples away`. This time the model produces the following equations:
```
 b = a + 3,  c = b / 2,  c = 13, a = ?
 ```
 and produces the correct answer `23.0`.

# Intro from the original repo
# Solving Math Word Problems by Combining Language Models With Symbolic Solvers

This is the repo for the paper: [Solving Math Word Problems by Combining Language Models With Symbolic Solvers](https://arxiv.org/pdf/2304.09102.pdf).

## Usage
To solve a math word problem, we first formalize the word problem as a set of variables and equations and then use `SymPy` to solve the equations.

Set the OpenAI key before running the script:
```export OPENAI_API_KEY='sk-...'```

```
from utils import *
from prompts.declarative_three_shot import DECLARATIVE_THREE_SHOT_AND_PRINCIPLES

question = 'xxx'
eq_list = get_declarative_equations(model='code-davinci-002', question=question, prompt=DECLARATIVE_THREE_SHOT_AND_PRINCIPLES, max_tokens=600, stop_token='\n\n\n', temperature=0)
answer = get_final_using_sympy(eq_list)
```

The `get_declarative_equations` method calls the OpenAI API to generate the equations, and the `get_final_using_sympy` method solves the equations to get the final answer (see an example in `run.py`).