
# Table of Contents

1.  [Legend](#org194be85)
    1.  [This means **definition**](#org514d250)
    2.  [Has been added to this script](#orgc7a86e6)
2.  [Documenting what works and what does not with go/unused-private-const-var-same-package](#org4616458)
    1.  [plan](#org1e17e49)
    2.  [problems](#org9dc8f9a)
    3.  [Iterations](#org3eed61d)
        1.  [1. This works](#org67c37e0)
        2.  [2. This works](#org386924e)
        3.  [3. This does not work](#orgc871267)
        4.  [4. This is inconsequential](#org01807ab)
        5.  [5. This does not work](#org08830b5)
    4.  [Testing hypothesis 1 (H1)](#org2590d96)
        1.  [Hypothesis 1 (H1)](#orgfccbd82)
        2.  [Experiment 1](#org1339839)
        3.  [Experiment 2](#orgfda3db7)
3.  [Collect data, look for new hypothesis](#org43b9e8e)
    1.  [Experiment 1](#orgb2e42ee)
        1.  [Plan](#org4b4e651)
        2.  [Modifications](#orga9a7e5b)
        3.  [Code](#org4628448)
        4.  [Possible hypothesis](#orge26a47b)
        5.  [Results](#org8e8c818)
4.  [What is the boolean logic I have to express to get what I want?](#orge985723)
    1.  [CLQL needs more clear boolean logic.](#orga9fbf8d)


<a id="org194be85"></a>

# Legend


<a id="org514d250"></a>

## This means **definition**

    go.value_spec:
      go.names:


<a id="orgc7a86e6"></a>

## Has been added to this script

[scripts/clql-annotate](file:///home/shane/notes2018/ws/codelingo/scripts/clql-annotate)


<a id="org4616458"></a>

# Documenting what works and what does not with [go/unused-private-const-var-same-package](file:///home/shane/var/smulliga/source/git/mullikine/codelingo/tenets/codelingo/go/unused-private-const-var-same-package/)


<a id="org1e17e49"></a>

## plan

First, get to something that works.
Then, try to add include and exclude. At each step, when it breaks, record it.


<a id="org9dc8f9a"></a>

## problems

The turnover rate is **very** slow running a review on a single tenet.


<a id="org3eed61d"></a>

## Iterations


<a id="org67c37e0"></a>

### 1. This works

1.  Modifications made (to get it to start working)

    1.  DONE `go.gen_decl(depth = 1):`
    
        Making depth = 1 here fixed something.
        
            vim +/"var b unexportedStructA" "/tmp/td_wdnSsHzB/temp_repo/example/foo.go"
    
    2.  TODO Temporarily removed `go.dir:`
    
        I can probably put this back in without problems.
    
    3.  DONE Removed `include:`
    
        Things started working when this was removed.

2.  Code

        import codelingo/ast/go
        
        # go.dir:
        go.file(depth = any):
          go.gen_decl(depth = 1):
            go.value_spec:
              go.names:
                @ review.comment
                go.ident:
                  @ review.vars.globalname
                  name as privateVarName
                  private == "true"
        
        # exclude:
        go.file(depth = any):
          exclude:
            #go.value_spec(depth = any):
            go.value_spec:
              go.names:
                # include:
                go.ident(depth = any):
                  name == privateVarName
                  private == "true"


<a id="org386924e"></a>

### 2. This works

1.  Modifications

    1.  Commented out `exclude:` &#x2026; `include:`.

2.  Code

        import codelingo/ast/go
        
        go.dir:
          go.file(depth = any):
            go.gen_decl(depth = 1):
              go.value_spec:
                go.names:
                  @ review.comment
                  go.ident:
                    @ review.vars.globalname
                    name as privateVarName
                    private == "true"
          exclude:
            go.file(depth = any):
              ## exclude:
              #go.value_spec(depth = any):
              #  # go.value_spec:
              #  go.names:
              #    include:
              go.ident(depth = any):
                name == privateVarName
                private == "true"


<a id="orgc871267"></a>

### 3. This does not work

1.  Modifications

    1.  Reinstated `include:`.

2.  Code

        import codelingo/ast/go
        
        go.dir:
          go.file(depth = any):
            go.gen_decl(depth = 1):
              go.value_spec:
                go.names:
                  @ review.comment
                  go.ident:
                    @ review.vars.globalname
                    name as privateVarName
                    private == "true"
          exclude:
            go.file(depth = any):
              ## exclude:
              go.value_spec(depth = any):
                # go.value_spec:
                go.names:
                  include:
                    go.ident(depth = any):
                      name == privateVarName
                      private == "true"

3.  Client Error

        Sorry, an error occurred while processing your request. Please try again.

4.  Server (platform) Error

        mullikine hit a uncaught error: result missing excluded parents
        /home/dev/projects/src/github.com/codelingo/platform/controller/graphdb/query/processor/validator/excluded_variable.go:247: result missing excluded parents


<a id="org01807ab"></a>

### 4. This is inconsequential

1.  Modifications

    1.  Removed depth = any
    
            29c29
            <                   go.ident(depth = any):
            ---
            >                   go.ident:

2.  Code

        import codelingo/ast/go
        
        go.dir:
          go.file(depth = any):
            go.gen_decl(depth = 1):
              go.value_spec:
                go.names:
                  @ review.comment
                  go.ident:
                    @ review.vars.globalname
                    name as privateVarName
                    private == "true"
          exclude:
            go.file(depth = any):
              ## exclude:
              go.value_spec(depth = any):
                # go.value_spec:
                go.names:
                  include:
                    go.ident:
                      name == privateVarName
                      private == "true"


<a id="org08830b5"></a>

### 5. This does not work

Same platform error as before: `result missing excluded parents`.

1.  Modifications

    1.  Swapped order of `exclude:` and sibling `go.file:`

2.  Code

        import codelingo/ast/go
        
        go.dir:
          exclude:
            go.file(depth = any):
              go.value_spec(depth = any):
                go.names:
                  include:
                  @review.comment
                    go.ident(depth = any):
                      name == privateVarName # But can I used a variable that was named later? A question for another time.
                      private == "true"
          go.file(depth = any):
            go.gen_decl(depth = 1):
              go.value_spec:
                go.names:
                  @ review.comment
                  go.ident:
                    @ review.vars.globalname
                    name as privateVarName
                    private == "true"

3.  Client Error

        Sorry, an error occurred while processing your request. Please try again.

4.  Server (platform) Error

        mullikine hit a uncaught error: result missing excluded parents
        /home/dev/projects/src/github.com/codelingo/platform/controller/graphdb/query/processor/validator/excluded_variable.go:247: result missing excluded parents

5.  Hypothesis 1

    An `exclude:` that has descendent `include:` can only belong as a child of its own parent by itself; ie. the `exclude:` must not have any siblings.
    
    1.  Are there any other examples of tenets with `exclude:` that has siblings?
    
        1.  In php, yes.
        
            [offsite-redirection/codelingo.yaml](file:///home/shane/var/smulliga/source/git/mullikine/codelingo/tenets/codelingo/php/offsite-redirection/codelingo.yaml)
        
        2.  In go, yes, so long as it doesn't have a parent `go.dir:`.
        
            [tested/codelingo.yaml](file:///home/shane/var/smulliga/source/git/mullikine/codelingo/tenets/codelingo/go/tested/codelingo.yaml)
        
        3.  Perhaps the problem is that an `exclude:` with an `include:` must not have any siblings.
        
                go.for_stmt:
                  exclude:
                    go.assign_stmt: # TODO: repeated index-of-array macrofact
                      go.rhs:
                        go.unary_expr:
                          go.index_expr:
                            go.selector_expr:
                              go.ident: # refers to lhs
                                name == structName
                              go.ident:
                                name == fieldName
                  @review.comment
                  go.call_expr(depth = any):
                    go.selector_expr:
                      go.ident:
                        name == structName
                      go.ident:
                        name == updatingFunction
        
        4.  This is confusing, but it appears that `exclude:` can have siblings and can have nested `exclude:`.
        
            The 2nd exclude here is acting as a complement of the set. Maybe it should be called 'complement'.
            
                # include functions with any "return nil"
                go.return_stmt(depth = any):
                  go.results:
                    go.ident:
                      name == "nil"
                
                # exclude functions with anything but "return nil"
                exclude:
                  go.return_stmt(depth = any):
                    go.results:
                      exclude:
                        go.ident:
                          name == "nil"
            
                cd "$HOME$MYGIT/mullikine/codelingo/tenets/cacophony/default/defer-in-loop"; ead "include[:(]"
            
            Here is one example of how an include is to be used. But here, the `exclude:` does not have any siblings.
            
                go.for_stmt(depth = any):
                  exclude:
                    go.func_lit(depth = any):
                      include(depth = any):
                        @ review.comment
                        go.defer_stmt(depth = any)


<a id="org2590d96"></a>

## Testing hypothesis 1 (H1)


<a id="orgfccbd82"></a>

### Hypothesis 1 (H1)

An `exclude:` that has descendent `include:` can only belong as a child of its own parent by itself; ie. the `exclude:` must not have any siblings.


<a id="org1339839"></a>

### Experiment 1

1.  Modifications

    1.  Simplified

2.  Code

        import codelingo/ast/go
        
        go.dir:
          exclude:
            go.file(depth = any):
              go.value_spec(depth = any):
                go.names
                # include:
                #   go.ident(depth = any)
          go.file(depth = any):
            go.ident(depth = any):
              @ review.vars.globalname
              private == "true"

3.  Results

    No issues found.
    
    I didn't have a `@review.comment` but it didn't have any matches so it didn't complain.


<a id="orgfda3db7"></a>

### Experiment 2

1.  Modifications

    1.  Reinstated `include:`

2.  Code

        import codelingo/ast/go
        
        go.dir:
          exclude:
            go.file(depth = any):
              go.value_spec(depth = any):
                go.names:
                  include:
                    @review.comment
                    go.ident(depth = any)
          go.file(depth = any):
            go.ident(depth = any):
              @ review.vars.globalname
              private == "true"

3.  Results

    This ran successfully.
    It matched every single identifier though.
    To get the error back I need to add more specificity to the `go.file...go.ident` which is not in an `exclude:`.
    H1 is a bust.


<a id="org43b9e8e"></a>

# Collect data, look for new hypothesis


<a id="orgb2e42ee"></a>

## Experiment 1


<a id="org4b4e651"></a>

### Plan

Add specificity. Remove specificity gradually. This may elicit H2.


<a id="orga9a7e5b"></a>

### Modifications

1.  Add specificity.


<a id="org4628448"></a>

### Code

    import codelingo/ast/go
    
     go.dir:
       exclude:
         go.file(depth = any):
           go.value_spec(depth = any):
             go.names:
               include:
                 go.ident(depth = any)
       go.file(depth = any):
         go.gen_decl(depth = 1):
           go.value_spec:
             go.names:
               @ review.comment
               go.ident:
                 @ review.vars.globalname
                 private == "true"


<a id="orge26a47b"></a>

### Possible hypothesis

An `exclude:` with an `include:` will fail unless there is a guaranteed match already from a sibling to `exclude:`.


<a id="org8e8c818"></a>

### Results

This worked. So what is different to the thing that failed?

1.  Comparison

    1.  Failed (exhibit 1)
    
            import codelingo/ast/go
            
            go.dir:
              go.file(depth = any):
                go.gen_decl(depth = 1):
                  go.value_spec:
                    go.names:
                      @ review.comment
                      go.ident:
                        @ review.vars.globalname
                        name as privateVarName
                        private == "true"
              exclude:
                go.file(depth = any):
                  ## exclude:
                  go.value_spec(depth = any):
                    # go.value_spec:
                    go.names:
                      include:
                        go.ident(depth = any):
                          name == privateVarName
                          private == "true"
        
        1.  Client Error
        
                Sorry, an error occurred while processing your request. Please try again.
        
        2.  Server (platform) Error
        
                mullikine hit a uncaught error: result missing excluded parents
                /home/dev/projects/src/github.com/codelingo/platform/controller/graphdb/query/processor/validator/excluded_variable.go:247: result missing excluded parents
    
    2.  Worked (exhibit 2)
    
            import codelingo/ast/go
            
            go.dir:
              exclude:
                go.file(depth = any):
                  go.value_spec(depth = any):
                    go.names:
                      include:
                        go.ident(depth = any)
              go.file(depth = any):
                go.gen_decl(depth = 1):
                  go.value_spec:
                    go.names:
                      @ review.comment
                      go.ident:
                        @ review.vars.globalname
                        private == "true"
        
        This matched only a few things. I think it matched correctly. Verify.
        This would be evidence for `exclude:` `include:` needing to come before an `exclude:` sibling.
        
        1.  Does it do what the tenet needs it to do?
        
            `{{globalname}}` is not matched.
    
    3.  Possibly working, test (exhibit 3)
    
        I haven't changed anything here, just reordered `exclude:` and sibling `go.file:`.
        
            import codelingo/ast/go
            
            go.dir:
              go.file(depth = any):
                go.gen_decl(depth = 1):
                  go.value_spec:
                    go.names:
                      @ review.comment
                      go.ident:
                        @ review.vars.globalname
                        private == "true"
              exclude:
                go.file(depth = any):
                  go.value_spec(depth = any):
                    go.names:
                      include:
                        go.ident(depth = any)
        
        1.  Conclusion
        
            It failed.
            
                flow config: failed to build issue, 'review.comment' not found in result
            
            So it appears that here, `exclude: include:` matched something yet does not have a `@ review.comment`.
            This makes sense.
            However, its behavior is different from last time, which does not make sense. I have only reordered.
        
        2.  Conclusion 1
        
            Order of exclude: matters. Not sure what the difference is behind the scenes.
            Should a particular ordering be enforced on the CLQL server-side?
            
            1.  As a side effect, it affects the `@review.comment` decorator
    
    4.  Possibly working, test (exhibit 4)
    
        Move the `@ review.comment` to see if I can get it to run without failing, changing as little code as possible.
        
        1.  code which is matching everything &#x2013; everything or all variables? verify &#x2013; yes, absolutely everything
        
                import codelingo/ast/go
                
                go.dir:
                  go.file(depth = any):
                    go.gen_decl(depth = 1):
                      go.value_spec:
                        go.names:
                          go.ident:
                            @ review.vars.globalname
                            private == "true"
                  exclude:
                    go.file(depth = any):
                      go.value_spec(depth = any):
                        go.names:
                          include:
                            @ review.comment
                            go.ident(depth = any)
        
        2.  It matched everything.
        
            So have a think about what is going on.
            Firstly, this might actually be the correct form (i.e. having exclude be the 2nd sibling).
            Secondly, it's excluding nothing at all because the thing being included (ie. `value_spec`) is always there.
            I have to make my non- `exclude:` `go.file` capture more things than the `exclude:` go.file.
            
            1.  What is unique, to go inside the `include:`?
            
                `go.value_spec:` is.
                
                Therefore, I must exclude those which do not have `go.value_spec`.
                
                1.  verbose identifier declaration at the top level of a file
                
                        import codelingo/ast/go
                        
                        go.file(depth = any):
                          go.decls:
                            go.gen_decl:
                              go.value_spec:
                                go.names:
                                  @playground.highlight
                                  go.ident:
                                    child_count == 0
                                    name == "privateVarA"
                                    private == "true"
                                    public == "false"
                                    type == "string"
                                    value == "\"baz\""
                
                2.  verbose identifier usage within a function call, printf
                
                        import codelingo/ast/go
                        
                        go.file(depth = any):
                          go.decls:
                            go.func_decl:
                              go.block_stmt:
                                go.list:
                                  go.expr_stmt:
                                    go.call_expr:
                                      go.args:
                                        @playground.highlight
                                        go.ident:
                                          child_count == 0
                                          name == "privateVarA"
                                          private == "true"
                                          public == "false"
                                          type == "string"
                                          value == "\"baz\""
        
        3.  More specified code
        
            1.  WARNING: on this test, I broke the CLQL by indenting the `exclude:`
            
                Ignore the conclusions made.
            
            2.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              @ review.comment
                              go.ident:
                                @ review.vars.globalname
                                private == "true"
                        exclude:
                          go.file(depth = any):
                            go.value_spec(depth = any):
                              go.names:
                                include:
                                  go.ident(depth = any)
        
        4.  Conclusion 2
        
            I need a better explanation of what the facts do. It's too confusing not having tooltips.
            Things like go.value<sub>spec</sub>.
            It's not sufficient to merely copy from the playground.
        
        5.  another test &#x2013; matches everything
        
                import codelingo/ast/go
                
                go.dir:
                  # this will match everything, even package. i only want to match variable definition
                  exclude:
                    go.file(depth = any):
                      go.value_spec(depth = any):
                        go.names:
                          include:
                            @ review.comment
                            go.ident(depth = any)
        
        6.  another test &#x2013; still matches everything
        
            1.  WARNING: This is using the indented `exclude:`, as before.
            
                I have been doing `file exclude file`&#x2026;
            
            2.  Code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              go.ident: # I may need to move ~@review.comment~ here
                                @ review.vars.globalname
                                private == "true"
                        exclude:
                          go.file(depth = any):
                            go.value_spec(depth = any):
                              go.names:
                                include:
                                  @ review.comment
                                  go.ident(depth = any)
        
        7.  remove `file exclude file`. rewrite how I think it should be
        
            1.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              go.ident: # I may need to move ~@review.comment~ here
                                @ review.vars.globalname
                                private == "true"
                      exclude:
                        go.file(depth = any):
                          go.value_spec(depth = any):
                            go.names:
                              include:
                                @ review.comment
                                go.ident(depth = any)
            
            2.  So what does this do?
            
                Matches every single thing.
                Remember that `value_spec` only appears on the definition of a `var/const`. &#x2013; This needs to be annotated.
        
        8.  try, try again. get this to do the thing the tenet wants
        
            1.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      # Match all top level definitions [which do is the 2nd part]. This part is definitely correct.
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              @ review.comment
                              go.ident: # I may need to move ~@review.comment~ here
                                private == "true"
                                @ review.vars.globalname
                                name as privateVarName
                    
                      # Exclude all usages
                      exclude:
                        # Match all usages
                        exclude:
                          go.file(depth = any):
                            go.value_spec(depth = any):
                              go.names:
                                include:
                                  go.ident(depth = any):
                                    name == privateVarName
            
            2.  So what does this do?
            
                Gives me that nasty error.
                
                1.  Client Error
                
                        Sorry, an error occurred while processing your request. Please try again.
                
                2.  Server (platform) Error
                
                        mullikine hit a uncaught error: result missing excluded parents
                        /home/dev/projects/src/github.com/codelingo/platform/controller/graphdb/query/processor/validator/excluded_variable.go:247: result missing excluded parents
        
        9.  try again. This time match the opposite of what the tenet wants
        
            Only one `exclude:`
            
            1.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      # Match all top level definitions [which do is the 2nd part]. This part is definitely correct.
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              @ review.comment
                              go.ident: # I may need to move ~@review.comment~ here
                                private == "true"
                                @ review.vars.globalname
                                name as privateVarName
                    
                      # Match all usages
                      exclude:
                        go.file(depth = any):
                          go.value_spec(depth = any):
                            go.names:
                              include:
                                go.ident(depth = any):
                                  name == privateVarName
            
            2.  So what does this do?
            
                Still gives me that nasty error.
        
        10. try again. again try to match the opposite of what the tenet wants
        
            Only one `exclude:`
            
            1.  modifications
            
                1.  swapped ordering of `exclude:` and `go:file`
            
            2.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      # Match all usages
                      exclude:
                        go.file(depth = any):
                          go.value_spec(depth = any):
                            go.names:
                              include:
                                go.ident(depth = any):
                                  name == privateVarName
                    
                      # Match all top level definitions [which do is the 2nd part]. This part is definitely correct.
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              @ review.comment
                              go.ident: # I may need to move ~@review.comment~ here
                                private == "true"
                                @ review.vars.globalname
                                name # as privateVarName
            
            3.  I'm expecting different things to happen
            
                1.  The nasty error should not occur.
            
            4.  So what does this do?
            
                1.  Gives me the nasty error.
        
        11. try again. again try to match the opposite of what the tenet wants
        
            -   Only one `exclude:`
            -   Except this time, unquote the `exclude:`.
            
            1.  modifications
            
                1.  swapped ordering of `exclude:` and `go:file`
            
            2.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      # # Match all usages
                      # exclude:
                      #   go.file(depth = any):
                      #     go.value_spec(depth = any):
                      #       go.names:
                      #         include:
                      #           go.ident(depth = any):
                      #             name == privateVarName
                    
                      # Match all top level definitions [which do is the 2nd part]. This part is definitely correct.
                      go.file(depth = any):
                        go.gen_decl(depth = 1):
                          go.value_spec:
                            go.names:
                              @ review.comment
                              go.ident: # I may need to move ~@review.comment~ here
                                private == "true"
                                @ review.vars.globalname
                                name # as privateVarName
            
            3.  So what does this do?
            
                1.  This DOES match the top level definitions. It's correct.
                
                    So why does the exclude break things.
                    Now try the `exclude:` on its own.
        
        12. try again. again try to match the opposite of what the tenet wants
        
            -   Only one `exclude:`
            -   Except this time, unquote the `exclude:`.
            
            1.  modifications
            
                1.  swapped ordering of `exclude:` and `go:file`
            
            2.  code
            
                    import codelingo/ast/go
                    
                    go.dir:
                      # Match all *usages*. This must match usages *only*.
                      exclude:
                        go.file(depth = any):
                          go.value_spec(depth = any):
                            go.names:
                              include:
                                @ review.comment
                                go.ident(depth = any):
                                  name # == privateVarName
                    
                      # # Match all top level definitions [which do is the 2nd part]. This part is definitely correct.
                      # go.file(depth = any):
                      #   go.gen_decl(depth = 1):
                      #     go.value_spec:
                      #       go.names:
                      #         @ review.comment
                      #         go.ident: # I may need to move ~@review.comment~ here
                      #           private == "true"
                      #           @ review.vars.globalname
                      #           name # as privateVarName
            
            3.  So what does this do?
            
                1.  This DOES match the top level definitions. It's correct.
                
                    So why does the exclude break things.
                    Now try the `exclude:` on its own.


<a id="orge985723"></a>

# What is the boolean logic I have to express to get what I want?

It's difficult to express this in terms of `exclude:` and `include:`.
We need boolean macros.
Also `exclude:` should be order agnostic.


<a id="orga9fbf8d"></a>

## CLQL needs more clear boolean logic.

