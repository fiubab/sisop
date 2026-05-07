#sisop

---

Aging is a technique to prevent starvation when implementing a priority scheduler for jobs. In this course context we refer to jobs as processes or environments (jos).

Its goal is to provide a fair execution time in the cpu for all the envs.

It consists in gradually incrementing the priority of the less executed jobs.
Incrementing priority doesnt has to be literal as it is, you can simply calculate prior based on the execution times number with this formula:

```C
effective_priority = env->priority - (env->env_runs / AGING_FACTOR);
```

Real prior less a number that grows based on the number of runs and AGING FACTOR defines how much runs start to affect that real prior.