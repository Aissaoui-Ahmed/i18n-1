# Pull Requests

Hay dos componentes fundamentales del proceso de Pull Request: uno concreto y técnico, y uno más orientado al proceso. El componente concreto y técnico implica los detalles específicos de la configuración de tu entorno local para que puedas realizar los cambios reales. Aquí es donde comenzaremos.

* [Dependencias](#dependencies)
* [Configurando su entorno local](#setting-up-your-local-environment) 
  * [Paso 1: Bifurcación](#step-1-fork)
  * [Paso 2: Branch](#step-2-branch)
* [El proceso de hacer cambios](#the-process-of-making-changes) 
  * [Paso 3: Código](#step-3-code)
  * [Paso 4: Commit](#step-4-commit) 
    * [Pautas del mensaje Commit](#commit-message-guidelines)
  * [Paso 5: Rebase](#step-5-rebase)
  * [Paso 6: Prueba](#step-6-test) 
    * [Cobertura de prueba](#test-coverage)
  * [Paso 7: Push](#step-7-push)
  * [Paso 8: Apertura de la Pull Request](#step-8-opening-the-pull-request)
  * [Paso 9: Discutir y actualizar](#step-9-discuss-and-update) 
    * [Flujo de trabajo de aprobación y cambios de solicitud](#approval-and-request-changes-workflow)
  * [Paso 10: Ejecutar](#step-10-landing)
* [Revisión de Pull Requests](#reviewing-pull-requests) 
  * [Revise un poco a la vez](#review-a-bit-at-a-time)
  * [Tenga en cuenta a la persona detrás del código](#be-aware-of-the-person-behind-the-code)
  * [Respete el tiempo de espera mínimo para los comentarios](#respect-the-minimum-wait-time-for-comments)
  * [Pull Requests abandonadas o paralizadas](#abandoned-or-stalled-pull-requests)
  * [Aprobar un cambio](#approving-a-change)
  * [Acepte que hay opiniones diferentes sobre lo que pertenece a Node.js](#accept-that-there-are-different-opinions-about-what-belongs-in-nodejs)
  * [El rendimiento no es todo](#performance-is-not-everything)
  * [Pruebas de integración continua](#continuous-integration-testing)
* [Notas adicionales](#additional-notes) 
  * [Commit Squashing](#commit-squashing)
  * [Obtener aprobaciones para su Pull Request](#getting-approvals-for-your-pull-request)
  * [Prueba de CI](#ci-testing)
  * [Esperar hasta que se cierre la Pull Request](#waiting-until-the-pull-request-gets-landed)
  * [Consulte la Guía del Colaborador](#check-out-the-collaborator-guide)

## Dependencias

Node.js tiene varias dependencias empaquetadas en las *deps/* y en los directorios de *tools/* que no son parte del proyecto propiamente dicho. Los cambios a los archivos en esos directorios deben enviarse a sus respectivos proyectos. No envíe un parche a Node.js. No podemos aceptar dichos parches.

En caso de duda, abra un problema en el [issue tracker](https://github.com/nodejs/node/issues/) o contacte a uno de los [colaboradores del proyecto](https://github.com/nodejs/node/#current-project-team-members). Node.js tiene dos canales de IRC: [#Node.js](https://webchat.freenode.net/?channels=node.js), para ayuda general y preguntas, y [#Node-dev](https://webchat.freenode.net/?channels=node-dev), específicamente para el desarrollo del núcleo de Node.js.

## Configurando su entorno local

Para empezar, necesitará tener `git` instalado de forma local. Dependiendo de su sistema operativo, también hay una serie de otras dependencias requeridas. Estas se detallan en la [Building guide](../../../BUILDING.md).

Una vez que tenga `git` y esté seguro de tener todas las dependencias necesarias, es hora de crear un fork.

### Paso 1: Bifurcación

Bifurque el proyecto [en GitHub](https://github.com/nodejs/node) y clone su bifurcación de forma local.

```text
$ git clone git@github.com:username/node.git
$ cd node
$ git remote add upstream https://github.com/nodejs/node.git
$ git fetch upstream
```

Se recomienda configurar `git` para que sepa quién es usted:

```text
$ git config user.name "J. Random User"
$ git config user.email "j.random.user@example.com"
```

Asegúrate de que este correo electrónico local también se haya agregado a tu [lista de correo electrónico de GitHub](https://github.com/settings/emails), de modo que tus commits se asocien correctamente con tu cuenta y seas promovido a Contribuyente una vez que se haya descargado tu primer commit.

### Paso 2: Branch

Como práctica recomendada para mantener su entorno de desarrollo lo más organizado posible, cree branches locales para trabajar dentro de ellas. Estos también se deben crear directamente fuera del branch `principal`.

```text
$ git checkout -b my-branch -t upstream/master
```

## El proceso de hacer cambios

### Paso 3: Código

La gran mayoría de las Pull Requests abiertas en el repositorio `nodejs/node` incluye cambios en uno o más de los siguientes:

     - el código C/C ++ contenido en el directorio `src`
     - el código JavaScript contenido en el directorio `lib`
     - la documentación en `doc / api`
     - pruebas dentro del directorio `test`.
    

Si está modificando el código, por favor asegúrese de ejecutar `make lint` de vez en cuando para asegurarse de que los cambios sigan la guía de estilo del código de Node.js.

Cualquier documentación que escriba (incluidos los comentarios de código y la documentación de API) debe seguir la [Guía de Estilo](../../STYLE_GUIDE.md). Las muestras de código incluidos en los documentos API también se verificarán cuando se ejecute `make lint` (o `vcbuild.bat lint`, en Windows).

Para contribuir con el código de C++, es posible que desee consultar la [Guía de Estilo de C++](../../../CPP_STYLE_GUIDE.md).

### Paso 4: Commit

Es una mejor práctica recomendada mantener sus cambios lo más lógicamente agrupados dentro de los commits individuales. No hay límite para el número de commits que puede tener una sola Pull Request, y a muchos colaboradores les resulta más fácil revisar los cambios que se dividen en varios commits.

```text
$ git add my/changed/files
$ git commit
```

Tenga en cuenta que los commits múltiples a menudo se reducen cuando se aterrizan (consulte las notas sobre [commit squashing](#commit-squashing)).

#### Pautas del mensaje Commit

Un buen mensaje de commit debe describir qué cambió y por qué.

1. La primera línea debe:
  
  * contener una breve descripción del cambio (preferiblemente 50 caracteres o menos, y no más de 72 caracteres)
  * estar completamente en minúsculas, con la excepción de los nombres propios, acrónimos y las palabras que hacen referencia al código, como nombres de funciones/variables
  * ir precedido del nombre del subsistema modificado y comenzar con un verbo imperativo. Comprueba la salida de `git log --oneline files/you/changed` para averiguar qué subsistemas tocan tus cambios.
    
    Ejemplos (en inglés):
  
  * `net: add localAddress and localPort to Socket`
  
  * `src: fix typos in async_wrap.h`

2. Mantenga la segunda línea en blanco.

3. Envuelva todas las otras líneas en 72 columnas (a excepción de las URL largas).

4. Si su parche soluciona un problema abierto, puede agregar una referencia al final del registro. Utilice el prefijo `Fixes:` y la URL completa del problema. Para otras referencias, use `Refs:`.
  
  Ejemplos:
  
  * `Fixes: https://github.com/nodejs/node/issues/1337`
  * `Refs: http://eslint.org/docs/rules/space-in-parens.html`
  * `Refs: https://github.com/nodejs/node/pull/3615`

5. Si su commit introduce un cambio de ruptura(`semver-major`), debe contener una explicación sobre el motivo del cambio de ruptura, qué situación desencadenaría el cambio de ruptura y cuál es el cambio exacto.

Muestra de mensaje de confirmación completo (en inglés):

```txt
subsystem: explain the commit in one line

Body of commit message is a few lines of text, explaining things
in more detail, possibly giving some background about the issue
being fixed, etc.

The body of the commit message can be several paragraphs, and
please do proper word-wrap and keep columns shorter than about
72 characters or so. That way, `git log` will show things
nicely even when it is indented.

Fixes: https://github.com/nodejs/node/issues/1337
Refs: http://eslint.org/docs/rules/space-in-parens.html
```

Si eres nuevo en contribuir con Node.js, por favor intenta hacer tu mejor esfuerzo para cumplir con estas pautas, pero no te preocupes si haces algo mal. Uno de los colaboradores actuales ayudará a ubicar las cosas y el contribuyente que aterrice la Solicitud de extracción garantizará que todo siga las pautas del proyecto.

Vea [core-validate-commit](https://github.com/evanlucas/core-validate-commit) - Una utilidad que asegura que los commits sigan las pautas de formato de commit.

### Paso 5: Rebase

Como buena práctica, una vez que haya confirmado sus cambios, es una buena idea usar `git rebase` (no `git merge`) para sincronizar su trabajo con el repositorio principal.

```text
$ git fetch upstream
$ git rebase upstream/master
```

Esto garantiza que su branch de trabajo tenga los últimos cambios del maestro `nodejs/node`.

### Paso 6: Prueba

Las correcciones de errores y las características siempre deben venir con las pruebas. Se ha proporcionado una [guía para escribir pruebas en Node.js](../writing-tests.md) para facilitar el proceso. Ver otras pruebas para ver cómo deberían estructurarse también puede ayudar.

El directorio `test` dentro del repositorio `nodejs/node` es complejo y, a menudo, no está claro a dónde debe ir un nuevo archivo de prueba. When in doubt, add new tests to the `test/parallel/` directory and the right location will be sorted out later.

Before submitting your changes in a Pull Request, always run the full Node.js test suite. To run the tests (including code linting) on Unix / macOS:

```text
$ ./configure && make -j4 test
```

And on Windows:

```text
> vcbuild test
```

(See the [Building guide](../../../BUILDING.md) for more details.)

Make sure the linter does not report any issues and that all tests pass. Please do not submit patches that fail either check.

If you want to run the linter without running tests, use `make lint`/`vcbuild lint`. It will run both JavaScript linting and C++ linting.

If you are updating tests and just want to run a single test to check it:

```text
$ python tools/test.py -J --mode=release parallel/test-stream2-transform
```

You can execute the entire suite of tests for a given subsystem by providing the name of a subsystem:

```text
$ python tools/test.py -J --mode=release child-process
```

If you want to check the other options, please refer to the help by using the `--help` option

```text
$ python tools/test.py --help
```

You can usually run tests directly with node:

```text
$ ./node ./test/parallel/test-stream2-transform.js
```

Remember to recompile with `make -j4` in between test runs if you change code in the `lib` or `src` directories.

#### Test Coverage

It's good practice to ensure any code you add or change is covered by tests. You can do so by running the test suite with coverage enabled:

```text
$ ./configure --coverage && make coverage
```

A detailed coverage report will be written to `coverage/index.html` for JavaScript coverage and to `coverage/cxxcoverage.html` for C++ coverage.

*Note that generating a test coverage report can take several minutes.*

To collect coverage for a subset of tests you can set the `CI_JS_SUITES` and `CI_NATIVE_SUITES` variables:

```text
$ CI_JS_SUITES=child-process CI_NATIVE_SUITES= make coverage
```

The above command executes tests for the `child-process` subsystem and outputs the resulting coverage report.

Running tests with coverage will create and modify several directories and files. To clean up afterwards, run:

```text
make coverage-clean
./configure && make -j4.
```

### Step 7: Push

Once you are sure your commits are ready to go, with passing tests and linting, begin the process of opening a Pull Request by pushing your working branch to your fork on GitHub.

```text
$ git push origin my-branch
```

### Step 8: Opening the Pull Request

From within GitHub, opening a new Pull Request will present you with a template that should be filled out:

```markdown
<!--
Thank you for your Pull Request. Please provide a description above and review
the requirements below.

Bug fixes and new features should include tests and possibly benchmarks.

Contributors guide: https://github.com/nodejs/node/blob/master/CONTRIBUTING.md
-->

#### Checklist
<!-- Remove items that do not apply. For completed items, change [ ] to [x]. -->

- [ ] `make -j4 test` (UNIX), or `vcbuild test` (Windows) passes
- [ ] tests and/or benchmarks are included
- [ ] documentation is changed or added
- [ ] commit message follows [commit guidelines](https://github.com/nodejs/node/blob/master/doc/guides/contributing/pull-requests.md#commit-message-guidelines)
```

Please try to do your best at filling out the details, but feel free to skip parts if you're not sure what to put.

Once opened, Pull Requests are usually reviewed within a few days.

### Step 9: Discuss and update

You will probably get feedback or requests for changes to your Pull Request. This is a big part of the submission process so don't be discouraged! Some contributors may sign off on the Pull Request right away, others may have more detailed comments or feedback. This is a necessary part of the process in order to evaluate whether the changes are correct and necessary.

To make changes to an existing Pull Request, make the changes to your local branch, add a new commit with those changes, and push those to your fork. GitHub will automatically update the Pull Request.

```text
$ git add my/changed/files
$ git commit
$ git push origin my-branch
```

It is also frequently necessary to synchronize your Pull Request with other changes that have landed in `master` by using `git rebase`:

```text
$ git fetch --all
$ git rebase origin/master
$ git push --force-with-lease origin my-branch
```

**Important:** The `git push --force-with-lease` command is one of the few ways to delete history in `git`. Before you use it, make sure you understand the risks. If in doubt, you can always ask for guidance in the Pull Request or on [IRC in the #node-dev channel](https://webchat.freenode.net?channels=node-dev&uio=d4).

If you happen to make a mistake in any of your commits, do not worry. You can amend the last commit (for example if you want to change the commit log).

```text
$ git add any/changed/files
$ git commit --amend
$ git push --force-with-lease origin my-branch
```

There are a number of more advanced mechanisms for managing commits using `git rebase` that can be used, but are beyond the scope of this guide.

Feel free to post a comment in the Pull Request to ping reviewers if you are awaiting an answer on something. If you encounter words or acronyms that seem unfamiliar, refer to this [glossary](https://sites.google.com/a/chromium.org/dev/glossary).

#### Approval and Request Changes Workflow

All Pull Requests require "sign off" in order to land. Whenever a contributor reviews a Pull Request they may find specific details that they would like to see changed or fixed. These may be as simple as fixing a typo, or may involve substantive changes to the code you have written. While such requests are intended to be helpful, they may come across as abrupt or unhelpful, especially requests to change things that do not include concrete suggestions on *how* to change them.

Try not to be discouraged. If you feel that a particular review is unfair, say so, or contact one of the other contributors in the project and seek their input. Often such comments are the result of the reviewer having only taken a short amount of time to review and are not ill-intended. Such issues can often be resolved with a bit of patience. That said, reviewers should be expected to be helpful in their feedback, and feedback that is simply vague, dismissive and unhelpful is likely safe to ignore.

### Step 10: Landing

In order to land, a Pull Request needs to be reviewed and [approved](#getting-approvals-for-your-pull-request) by at least one Node.js Collaborator and pass a [CI (Continuous Integration) test run](#ci-testing). After that, as long as there are no objections from other contributors, the Pull Request can be merged. If you find your Pull Request waiting longer than you expect, see the [notes about the waiting time](#waiting-until-the-pull-request-gets-landed).

When a collaborator lands your Pull Request, they will post a comment to the Pull Request page mentioning the commit(s) it landed as. GitHub often shows the Pull Request as `Closed` at this point, but don't worry. If you look at the branch you raised your Pull Request against (probably `master`), you should see a commit with your name on it. Congratulations and thanks for your contribution!

## Reviewing Pull Requests

All Node.js contributors who choose to review and provide feedback on Pull Requests have a responsibility to both the project and the individual making the contribution. Reviews and feedback must be helpful, insightful, and geared towards improving the contribution as opposed to simply blocking it. If there are reasons why you feel the PR should not land, explain what those are. Do not expect to be able to block a Pull Request from advancing simply because you say "No" without giving an explanation. Be open to having your mind changed. Be open to working with the contributor to make the Pull Request better.

Reviews that are dismissive or disrespectful of the contributor or any other reviewers are strictly counter to the [Code of Conduct](https://github.com/nodejs/admin/blob/master/CODE_OF_CONDUCT.md).

When reviewing a Pull Request, the primary goals are for the codebase to improve and for the person submitting the request to succeed. Even if a Pull Request does not land, the submitters should come away from the experience feeling like their effort was not wasted or unappreciated. Every Pull Request from a new contributor is an opportunity to grow the community.

### Review a bit at a time.

Do not overwhelm new contributors.

It is tempting to micro-optimize and make everything about relative performance, perfect grammar, or exact style matches. Do not succumb to that temptation.

Focus first on the most significant aspects of the change:

1. Does this change make sense for Node.js?
2. Does this change make Node.js better, even if only incrementally?
3. Are there clear bugs or larger scale issues that need attending to?
4. Is the commit message readable and correct? If it contains a breaking change is it clear enough?

When changes are necessary, *request* them, do not *demand* them, and do not assume that the submitter already knows how to add a test or run a benchmark.

Specific performance optimization techniques, coding styles and conventions change over time. The first impression you give to a new contributor never does.

Nits (requests for small changes that are not essential) are fine, but try to avoid stalling the Pull Request. Most nits can typically be fixed by the Node.js Collaborator landing the Pull Request but they can also be an opportunity for the contributor to learn a bit more about the project.

It is always good to clearly indicate nits when you comment: e.g. `Nit: change foo() to bar(). But this is not blocking.`

If your comments were addressed but were not folded automatically after new commits or if they proved to be mistaken, please, [hide them](https://help.github.com/articles/managing-disruptive-comments/#hiding-a-comment) with the appropriate reason to keep the conversation flow concise and relevant.

### Be aware of the person behind the code

Be aware that *how* you communicate requests and reviews in your feedback can have a significant impact on the success of the Pull Request. Yes, we may land a particular change that makes Node.js better, but the individual might just not want to have anything to do with Node.js ever again. The goal is not just having good code.

### Respect the minimum wait time for comments

There is a minimum waiting time which we try to respect for non-trivial changes, so that people who may have important input in such a distributed project are able to respond.

For non-trivial changes, Pull Requests must be left open for *at least* 48 hours during the week, and 72 hours on a weekend. In most cases, when the PR is relatively small and focused on a narrow set of changes, these periods provide more than enough time to adequately review. Sometimes changes take far longer to review, or need more specialized review from subject matter experts. When in doubt, do not rush.

Trivial changes, typically limited to small formatting changes or fixes to documentation, may be landed within the minimum 48 hour window.

### Abandoned or Stalled Pull Requests

If a Pull Request appears to be abandoned or stalled, it is polite to first check with the contributor to see if they intend to continue the work before checking if they would mind if you took it over (especially if it just has nits left). When doing so, it is courteous to give the original contributor credit for the work they started (either by preserving their name and email address in the commit log, or by using an `Author:` meta-data tag in the commit.

### Approving a change

Any Node.js core Collaborator (any GitHub user with commit rights in the `nodejs/node` repository) is authorized to approve any other contributor's work. Collaborators are not permitted to approve their own Pull Requests.

Collaborators indicate that they have reviewed and approve of the changes in a Pull Request either by using GitHub's Approval Workflow, which is preferred, or by leaving an `LGTM` ("Looks Good To Me") comment.

When explicitly using the "Changes requested" component of the GitHub Approval Workflow, show empathy. That is, do not be rude or abrupt with your feedback and offer concrete suggestions for improvement, if possible. If you're not sure *how* a particular change can be improved, say so.

Most importantly, after leaving such requests, it is courteous to make yourself available later to check whether your comments have been addressed.

If you see that requested changes have been made, you can clear another collaborator's `Changes requested` review.

Change requests that are vague, dismissive, or unconstructive may also be dismissed if requests for greater clarification go unanswered within a reasonable period of time.

If you do not believe that the Pull Request should land at all, use `Changes requested` to indicate that you are considering some of your comments to block the PR from landing. When doing so, explain *why* you believe the Pull Request should not land along with an explanation of what may be an acceptable alternative course, if any.

### Accept that there are different opinions about what belongs in Node.js

Opinions on this vary, even among the members of the Technical Steering Committee.

One general rule of thumb is that if Node.js itself needs it (due to historic or functional reasons), then it belongs in Node.js. For instance, `url` parsing is in Node.js because of HTTP protocol support.

Also, functionality that either cannot be implemented outside of core in any reasonable way, or only with significant pain.

It is not uncommon for contributors to suggest new features they feel would make Node.js better. These may or may not make sense to add, but as with all changes, be courteous in how you communicate your stance on these. Comments that make the contributor feel like they should have "known better" or ridiculed for even trying run counter to the [Code of Conduct](https://github.com/nodejs/admin/blob/master/CODE_OF_CONDUCT.md).

### Performance is not everything

Node.js has always optimized for speed of execution. If a particular change can be shown to make some part of Node.js faster, it's quite likely to be accepted. Claims that a particular Pull Request will make things faster will almost always be met by requests for performance [benchmark results](../writing-and-running-benchmarks.md) that demonstrate the improvement.

That said, performance is not the only factor to consider. Node.js also optimizes in favor of not breaking existing code in the ecosystem, and not changing working functional code just for the sake of changing.

If a particular Pull Request introduces a performance or functional regression, rather than simply rejecting the Pull Request, take the time to work *with* the contributor on improving the change. Offer feedback and advice on what would make the Pull Request acceptable, and do not assume that the contributor should already know how to do that. Be explicit in your feedback.

### Continuous Integration Testing

All Pull Requests that contain changes to code must be run through continuous integration (CI) testing at <https://ci.nodejs.org/>.

Only Node.js core Collaborators with commit rights to the `nodejs/node` repository may start a CI testing run. The specific details of how to do this are included in the new Collaborator [Onboarding guide](../../onboarding.md).

Ideally, the code change will pass ("be green") on all platform configurations supported by Node.js (there are over 30 platform configurations currently). This means that all tests pass and there are no linting errors. In reality, however, it is not uncommon for the CI infrastructure itself to fail on specific platforms or for so-called "flaky" tests to fail ("be red"). It is vital to visually inspect the results of all failed ("red") tests to determine whether the failure was caused by the changes in the Pull Request.

## Additional Notes

### Commit Squashing

In most cases, do not squash commits that you add to your Pull Request during the review process. When the commits in your Pull Request land, they may be squashed into one commit per logical change. Metadata will be added to the commit message (including links to the Pull Request, links to relevant issues, and the names of the reviewers). The commit history of your Pull Request, however, will stay intact on the Pull Request page.

For the size of "one logical change", [0b5191f](https://github.com/nodejs/node/commit/0b5191f15d0f311c804d542b67e2e922d98834f8) can be a good example. It touches the implementation, the documentation, and the tests, but is still one logical change. All tests should always pass when each individual commit lands on the master branch.

### Getting Approvals for Your Pull Request

A Pull Request is approved either by saying LGTM, which stands for "Looks Good To Me", or by using GitHub's Approve button. GitHub's Pull Request review feature can be used during the process. For more information, check out [the video tutorial](https://www.youtube.com/watch?v=HW0RPaJqm4g) or [the official documentation](https://help.github.com/articles/reviewing-changes-in-pull-requests/).

After you push new changes to your branch, you need to get approval for these new changes again, even if GitHub shows "Approved" because the reviewers have hit the buttons before.

### CI Testing

Every Pull Request needs to be tested to make sure that it works on the platforms that Node.js supports. This is done by running the code through the CI system.

Only a Collaborator can start a CI run. Usually one of them will do it for you as approvals for the Pull Request come in. If not, you can ask a Collaborator to start a CI run.

### Waiting Until the Pull Request Gets Landed

A Pull Request needs to stay open for at least 48 hours (72 hours on a weekend) from when it is submitted, even after it gets approved and passes the CI. This is to make sure that everyone has a chance to weigh in. If the changes are trivial, collaborators may decide it doesn't need to wait. A Pull Request may well take longer to be merged in. All these precautions are important because Node.js is widely used, so don't be discouraged!

### Consulte la Guía del Colaborador

If you want to know more about the code review and the landing process, see the [Collaborator Guide](../../../COLLABORATOR_GUIDE.md).