---
title: Using Github Copilot for Terraform code refactoring
published: true
description: How AI assistants make Terraform greater than before
tags: terraform
canonical_url: null
date: '2025-11-23T18:36:19Z'
cover_image: ./logo.png
id: 3051956
---
 

Terraform and OpenTofu are the only major IaC tools today that provide a fully deterministic, declarative, provider-agnostic, predictable execution plan across AWS + Azure + other cloud platforms.

However, the Terraform development experience is still close to what we had for JavaScript in 2008: we have syntax highlighting, basic static analysis, and basic code navigation. 
But the Terraform code refactoring still should be done manually. 
We still have no button "extract resources to the module" in JetBrains IDE or VS Code. 
Mainly because in the Terraform world, code refactoring should go together with state transformation.

By the end of 2024 in my team had written a lot of Terraform code. 
We have learned how to master infrastructure as code using Terraform, our expertise has grown up and the code written two years ago become ugly to us. 
But state transformations and the complexity of code change stopped us from starting to do refactoring on a large scale. For example, we have states with 6K resources.

We tried to use multiple ways to do state transformations:

Created Python scripts to generate Terraform moved blocks for the particular refactoring.

Tried to use Terraform file templates and construct object maps to pass old and new resource structure.

All these approaches worked, but required a lot of effort, especially for huge states and code base.

Everything changed when GitHub Copilot introduced Edits mode. We started actively using it for Terraform code refactoring, and even then, I realized that Copilot can solve our code quality issues faster.

GitHub Copilot Agents automated the refactoring now we use the following algorithms:

1. **Ask Copilot for the Terraform code changes.** Here is very important to limit the change scope and do refactoring fraction by fraction. Better to have multiple iterations for refactoring to not lose control on changes.
   
2. **The plan for the small changes is much easier to review.** The most frequent refactoring we do are to extract resources to the module and change the module structure.
   
3. **Run plan and save output to the file `terraform plan -no-color > plan.txt`.** We need the plan saved to add it to the context. In Agent mode Copilot also able to use shell commands like grep to analyze a plan.
   
4. **Generate plan summary and validate if plan meet refactoring requirements.** We use that step if we need to work with huge states. If the plan contains 500 changed resources, it is difficult to say if the change is valid. The plan summary allows for reviewing types of affected resources and correlating them with the initial change.
   
5. **Generate moved.tf file with moved blocks to avoid any changes in the next plan.** On this step, the agent can do multiple iterations for moving blocks generation and runing  the Terraform plan. I would recommend to review moved.tf file because in complex cases LLM could do a wrong correlation between resources and generate move valid from Terraform point of view, but not from the purpose. For example:

    ```hcl
    moved {
      from = aws_iam_user[1].team_two_user
      to   = module["team1"].aws_iam_user.user
    }
    moved {
      from = aws_iam_user[2].team_one_user
      to   = module["team2"].aws_iam_user.user
    }
    
    ```


## Final Thoughts

In 2025, Terraform remains the strongest cloud‑agnostic Infrastructure‑as‑Code tool available. Pulumi is easier to develop with, but it is not as deterministic as Terraform’s declarative model. Modern AI assistants like GitHub Copilot and Cursor dramatically accelerate safe refactoring and reduce the fear of touching legacy state. With the help of AI‑driven tooling, Terraform has become a more powerful IaC solution than at any previous point in its history.
