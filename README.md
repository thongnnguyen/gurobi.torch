# gurobi.torch • [ ![Build Status] [travis-image] ] [travis] [ ![License] [license-image] ] [license]

*Unofficial Gurobi Torch bindings.*

[travis-image]: https://travis-ci.org/bamos/gurobi.torch.png?branch=master
[travis]: http://travis-ci.org/bamos/gurobi.torch

[license-image]: http://img.shields.io/badge/license-Apache--2-blue.svg?style=flat
[license]: LICENSE

---

You may also be interested in:
+ Torch [ECOS](https://github.com/embotech/ecos) bindings at
  [bamos/ecos.torch](https://github.com/bamos/ecos.torch).
+ A Torch library for convex optimization that implements
  spectral projected gradient and projected gradient descent at
  [bamos/cvx-optim.torch](https://github.com/bamos/cvx-optim.torch).

# Installation

1. Set `GUROBI_HOME` in your shell's current session (with `export GUROBI_HOME=..`)
   and initialization file (`.bashrc` or `.zshrc`).
2. `luarocks make`

# Usage

The following solves the linear program

```
min  c'*x s.t. G*x <= h
```

## Linear Program

```lua
local gurobi = require 'gurobi'

local G = torch.Tensor{{-1, 1}, {-1, -1}, {0, -1}, {1, -2}}
local h = torch.Tensor{1.0, -2.0, 0.0, 4.0}
local c = torch.Tensor{2.0, 1.0}

local env = gurobi.loadenv("")
local model = gurobi.newmodel(env, "", c)
gurobi.addconstrs(model, G, 'LE', h)
local status, x = gurobi.solve(model)
print(x) -- Optimal x is [0.5, 1.5]

gurobi.free(env, model)
```

## Linear Program with Incrementally Added Constraints
```lua
local gurobi = require 'gurobi'

-- minimize y
-- subject to y >= x
--            y >= -x
--            y >= x + 1
local c = torch.Tensor{0.0, 1.0}
local G = torch.Tensor{{1, -1}, {-1, -1}, {1, -1}}
local h = torch.Tensor{0.0, 0.0, -1.0}

local env = gurobi.loadenv("")
local model = gurobi.newmodel(env, "", c)

local I = {{1,2}}
gurobi.addconstrs(model, G[I], 'LE', h[I])

local status, x = gurobi.solve(model)
print(x) -- Optimal at this point is [0, 0]

gurobi.addconstr(model, G[3], 'LE', h[3])
status, x = gurobi.solve(model)
print(x) -- Optimal at this point is [-0.5, 0.5]

gurobi.free(env, model)
```

## OpenMP Parallel Solves
```lua
local env = gurobi.loadenv("")
local model1 = gurobi.newmodel(env, "", c)
gurobi.addconstrs(model1, G, 'LE', h)

local model2 = gurobi.newmodel(env, "", c)
gurobi.addconstrs(model2, G, 'LE', h)

local status, xs = gurobi.solvePar({model1, model2})
```

# Tests

After installing the library with `luarocks`, our tests in
[test.lua](https://github.com/bamos/gurobi.torch/blob/master/test.lua)
can be run with `th test.lua`.

# Licensing

+ Gurobi is proprietary software.
+ The original code in this repository (the Gurobi bindings) is
  [Apache-licensed](https://github.com/bamos/gurobi.torch/blob/master/LICENSE).
