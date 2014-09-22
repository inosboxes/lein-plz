- [lein-plz](#lein-plz)
  - [Usage](#usage)
    - [Nicknames Example](#nicknames-example)
    - [Groups Example](#groups-example)
  - [Setup](#setup)
    - [Adding your own nicknames](#adding-your-own-nicknames)
    - [Adding your own groups](#adding-your-own-groups)
  - [Built-in nicknames](#built-in-nicknames)
  - [Possible Issues](#possible-issues)
    - [Use with lein-ancient](#use-with-lein-ancient)
  - [License](#license)

# lein-plz<a id="sec-1" name="sec-1"></a>

```
ScaRy, ScaRy GUy, BOinG!
GRaPEfRUit fallS!
ScaRy, Sick, BaRfy…
GO anD… la la la!
DinG! ScaRy!

-- Mr. Saturn
```

A Leiningen plugin for quickly adding dependencies to projects.

## Usage<a id="sec-1-1" name="sec-1-1"></a>

### Nicknames Example<a id="sec-1-1-1" name="sec-1-1-1"></a>

Suppose you want to add `clojurescript`, `core.async` and
`data.json` to a project. Maybe you know the latest versions
off-hand (or at least, some version). You still have to type out
the groupIds and &#x2013; it takes some typing.

It's possible to do this instead:

```sh
$ lein plz add core.async cljs data.json
```

### Groups Example<a id="sec-1-1-2" name="sec-1-1-2"></a>

Suppose you're about to build a webapp to advertise
`core.logic`. You've setup groups, so you only need to combine the
dependencies from the server and client.

```sh
$ lein plz add server-group client-group core.logic
```

The result:

```clojure
:dependencies [[org.clojure/clojure "1.6.0"]
               [http-kit "2.1.18"]
               [compojure "1.1.8"]
               [om "0.7.1"]
               [org.clojure/core.async "0.1.338.0-5c5012-alpha"]
               [org.clojure/clojurescript "0.0-2311"]
               [org.clojure/core.logic "0.8.8"]]
```

## Setup<a id="sec-1-2" name="sec-1-2"></a>

Add `[lein-plz "0.2.1"]` to the `:plugins` vector in your user
profile.

```clojure
{:user {:plugins [[cider/cider-nrepl "0.8.0-SNAPSHOT"]
                  [lein-plz "0.2.1"]
                  [slamhound "1.5.5"]]}}
```

### Adding your own nicknames<a id="sec-1-2-1" name="sec-1-2-1"></a>

`lein-plz` comes prepared with a fairly comprehensive list of fallback nicknames,
which you can see here, or by using the `lein plz list` command. You can filter
the list by adding an additional argument to the command, which is interpreted
as a regexp to display only the matching items:

```sh
lein plz list org.clojure/
```

=>

```
+----------------------------+----------------+
| Dependency                 | Nickname(s)    |
+----------------------------+----------------+
| org.clojure/core.async     | core.async     |
| org.clojure/core.contracts | core.contracts |
| org.clojure/core.logic     | core.logic     |
| org.clojure/core.typed     | core.typed     |
| org.clojure/core.unify     | core.unify     |
| org.clojure/core.match     | core.match     |
| org.clojure/core.cache     | core.cache     |
| org.clojure/core.memoize   | core.memoize   |
+----------------------------+----------------+
```

The list comes from the [CrossClj.info](http://crossclj.info/) web site, where each library with a
popularity score of three and higher is included.

If your favorite library is not present, or you'd like to use a more convenient
nickname to you, then fear not! You can add more nicknames by including your
custom dependency -> nickname map like this (Note: these are built-in nicknames,
so there's no need to include these in your own map):

```clojure
{org.clojure/clojure         #{"clojure" "clj"}
 org.clojure/clojurescript   #{"clojurescript" "cljs"}
 org.clojure/core.async      #{"core.async"}
 compojure                   #{"compojure"}
 hiccup                      #{"hiccup"}
 ring                        #{"ring"}}
```

where the symbols are the full dependency names, while each value is a set of
nicknames for that dependency.

`lein-plz` maintains a global map of dependencies, equivalent to:

```clojure
(merge fallback-nickname-map user-map-1 user-map-2 user-map-3 ...)
```

If the example map resides in `/home/stuartsierra/.plz/myplzmap.edn`, then
`/home/stuartsierra/.lein/profiles.clj` should be:

```clojure
{:user {:plugins [[cider/cider-nrepl "0.8.0-SNAPSHOT"]
                  [lein-plz "0.2.1"]
                  [slamhound "1.5.5"]]
        :plz ["/home/stuartsierra/.plz/myplzmap.edn"]}}
```

You can use more than one map:

```clojure
{:user {:plugins [[cider/cider-nrepl "0.8.0-SNAPSHOT"]
                  [lein-plz "0.2.1"]
                  [slamhound "1.5.5"]]
        :plz ["/home/stuartsierra/.plz/myplzmap.edn"
              "/home/stuartsierra/.plz/user-map-2.edn"
              "/home/stuartsierra/.plz/user-map-3.edn"]}}
```

In case of conflicts, the last map always overrides the previous ones, as you'd expect from a `merge`.

### Adding your own groups<a id="sec-1-2-2" name="sec-1-2-2"></a>

You can add collections of dependencies at a time using
groups. Create files containing edn maps such as these:

```clojure
;; server-group
;; /home/stuartsierra/.plz/server.edn
{http-kit                        #{"http-kit"}
 compojure                       #{"compojure"}}

;; client-group
;; /home/stuartsierra/.plz/client.edn
{om                              #{"om"}
 org.clojure/core.async          #{"core.async" "async"}
 org.clojure/clojurescript       #{"clojurescript" "cljs"}}
```

The dependencies in each map can be referenced by following their
filename with :as key and the group's name.

```clojure
{:user {:plugins [[cider/cider-nrepl "0.8.0-SNAPSHOT"]
                  [lein-plz "0.2.1"]
                  [slamhound "1.5.5"]]
        :plz [["/home/stuartsierra/.plz/server.edn" :as "server-group"]
              ["/home/stuartsierra/.plz/client.edn" :as "client-group"]
              ["/home/stuartsierra/.plz/myplzmap.edn"]]}}
```

```sh
$ lein plz add server-group client-group
```

The merge order in adding your own nicknames (See section ) is maintained. [The
wiki has a collection of groups for getting started](https://github.com/johnwalker/lein-plz/wiki/Groups). Feel free to
contribute your own groups to the wiki!

## Built-in nicknames<a id="sec-1-3" name="sec-1-3"></a>

These nicknames are built-in. User options take precedence over these.

```org
+--------------------------------------+-----------------------------------+
| Dependency                           | Nickname(s)                       |
+--------------------------------------+-----------------------------------+
| alandipert/interpol8                 | interpol8                         |
| aleph                                | aleph                             |
| am.ik/clj-gae-testing                | clj-gae-testing                   |
| amazonica                            | amazonica                         |
| analyze                              | analyze                           |
| ancient-clj                          | ancient-clj                       |
| arrows-extra                         | arrows-extra                      |
| auto-reload                          | auto-reload                       |
| autodoc                              | autodoc                           |
| aysylu/loom                          | loom                              |
| backtick                             | backtick                          |
| base64-clj                           | base64-clj                        |
| bond                                 | bond                              |
| bultitude                            | bultitude                         |
| bux                                  | bux                               |
| byte-streams                         | byte-streams                      |
| byte-transforms                      | byte-transforms                   |
| camel-snake-kebab                    | camel-snake-kebab                 |
| caribou/antlers                      | antlers                           |
| caribou/caribou-core                 | caribou-core                      |
| caribou/caribou-frontend             | caribou-frontend                  |
| caribou/caribou-plugin               | caribou-plugin                    |
| cascalog                             | cascalog                          |
| cc.qbits/hayt                        | hayt                              |
| cheshire                             | cheshire                          |
| cider/cider-nrepl                    | cider-nrepl                       |
| clansi                               | clansi                            |
| classlojure                          | classlojure                       |
| clatrix                              | clatrix                           |
| clauth                               | clauth                            |
| clip-test                            | clip-test                         |
| clj-assorted-utils                   | clj-assorted-utils                |
| clj-aws-s3                           | clj-aws-s3                        |
| clj-campfire                         | clj-campfire                      |
| clj-dbcp                             | clj-dbcp                          |
| clj-debug                            | clj-debug                         |
| clj-glob                             | clj-glob                          |
| clj-gui                              | clj-gui                           |
| clj-http                             | clj-http                          |
| clj-http-fake                        | clj-http-fake                     |
| clj-http-lite                        | clj-http-lite                     |
| clj-info                             | clj-info                          |
| clj-jgit                             | clj-jgit                          |
| clj-json                             | clj-json                          |
| clj-kafka                            | clj-kafka                         |
| clj-logging-config                   | clj-logging-config                |
| clj-native                           | clj-native                        |
| clj-oauth                            | clj-oauth                         |
| clj-obt                              | clj-obt                           |
| clj-pail                             | clj-pail                          |
| clj-pail-tap                         | clj-pail-tap                      |
| clj-pdf                              | clj-pdf                           |
| clj-redis                            | clj-redis                         |
| clj-simple-form/clj-simple-form-core | clj-simple-form-core              |
| clj-ssh                              | clj-ssh                           |
| clj-stacktrace                       | clj-stacktrace                    |
| clj-statsd                           | clj-statsd                        |
| clj-text-decoration                  | clj-text-decoration               |
| clj-time                             | clj-time                          |
| clj-toml                             | clj-toml                          |
| clj-tuple                            | clj-tuple                         |
| clj-unit                             | clj-unit                          |
| clj-wallhack                         | clj-wallhack                      |
| cljs-ajax                            | cljs-ajax                         |
| cljs-http                            | cljs-http                         |
| cljs-uuid                            | cljs-uuid                         |
| cljsbuild                            | cljsbuild                         |
| clojail                              | clojail                           |
| clojure-complete                     | clojure-complete                  |
| clojure-csv                          | clojure-csv                       |
| clojure-opennlp                      | clojure-opennlp                   |
| clojure-source                       | clojure-source                    |
| clojure-tools                        | clojure-tools                     |
| clojure.options                      | clojure.options                   |
| clojurewerkz/cassaforte              | cassaforte                        |
| clojurewerkz/neocons                 | neocons                           |
| clojurewerkz/ogre                    | ogre                              |
| clojurewerkz/quartzite               | quartzite                         |
| clojurewerkz/support                 | support                           |
| clojurewerkz/urly                    | urly                              |
| clout                                | clout                             |
| clucy                                | clucy                             |
| co.paralleluniverse/pulsar           | pulsar                            |
| codox                                | codox                             |
| codox-md                             | codox-md                          |
| codox/codox.core                     | codox.core                        |
| codox/codox.leiningen                | codox.leiningen                   |
| collection-check                     | collection-check                  |
| colorize                             | colorize                          |
| com.andrewmcveigh/plugin-jquery      | plugin-jquery                     |
| com.backtype/dfs-datastores          | dfs-datastores                    |
| com.birdseye-sw/dalap                | dalap                             |
| com.birdseye-sw/lein-dalap           | lein-dalap                        |
| com.cemerick/austin                  | austin                            |
| com.cemerick/clojurescript.test      | clojurescript.test                |
| com.cemerick/double-check            | double-check                      |
| com.cemerick/friend                  | friend                            |
| com.cemerick/piggieback              | piggieback                        |
| com.cemerick/pomegranate             | pomegranate                       |
| com.cemerick/pprng                   | pprng                             |
| com.damballa/abracad                 | abracad                           |
| com.flyingmachine/webutils           | webutils                          |
| com.jakemccrary/lein-test-refresh    | lein-test-refresh                 |
| com.keminglabs/cljx                  | cljx                              |
| com.keminglabs/singult               | singult                           |
| com.madeye.clojure.ampache/ampachedb | ampachedb                         |
| com.madeye.clojure.common/common     | common                            |
| com.mdrogalis/onyx                   | onyx                              |
| com.novemberain/langohr              | langohr                           |
| com.novemberain/monger               | monger                            |
| com.novemberain/pantomime            | pantomime                         |
| com.novemberain/validateur           | validateur                        |
| com.onekingslane.danger/clojure-c... | clojure-common-utils              |
| com.palletops/api-builder            | api-builder                       |
| com.palletops/discovery-api-runtime  | discovery-api-runtime             |
| com.palletops/git-crate              | git-crate                         |
| com.palletops/java-crate             | java-crate                        |
| com.palletops/pallet-common          | pallet-common                     |
| com.palletops/rbenv-crate            | rbenv-crate                       |
| com.palletops/upstart-crate          | upstart-crate                     |
| com.redhat.qe/jul.test.records       | jul.test.records                  |
| com.relaynetwork/clorine             | clorine                           |
| com.ryanmcg/incise-codox             | incise-codox                      |
| com.ryanmcg/incise-vm-layout         | incise-vm-layout                  |
| com.stuartsierra/component           | component                         |
| com.stuartsierra/dependency          | dependency                        |
| com.taoensso/carmine                 | carmine                           |
| com.taoensso/encore                  | encore                            |
| com.taoensso/faraday                 | faraday                           |
| com.taoensso/nippy                   | nippy                             |
| com.taoensso/sente                   | sente                             |
| com.taoensso/timbre                  | timbre                            |
| com.taoensso/tower                   | tower                             |
| com.velisco/tagged                   | tagged                            |
| com.xab/sanity                       | sanity                            |
| compliment                           | compliment                        |
| compojure                            | compojure                         |
| conch                                | conch                             |
| congomongo                           | congomongo                        |
| control                              | control                           |
| conveyor                             | conveyor                          |
| crate                                | crate                             |
| criterium                            | criterium                         |
| crypto-equality                      | crypto-equality                   |
| crypto-random                        | crypto-random                     |
| dar/container                        | container                         |
| datomic-schematode                   | datomic-schematode                |
| debug                                | debug                             |
| degel-clojure-utils                  | degel-clojure-utils               |
| dieter                               | dieter                            |
| dire                                 | dire                              |
| dk.ative/docjure                     | docjure                           |
| domina                               | domina                            |
| dorothy                              | dorothy                           |
| easyconf                             | easyconf                          |
| ego                                  | ego                               |
| enfocus                              | enfocus                           |
| enlive                               | enlive                            |
| environ                              | environ                           |
| expectations                         | expectations                      |
| factual/drake-interface              | drake-interface                   |
| fast-zip                             | fast-zip                          |
| fetch                                | fetch                             |
| figwheel                             | figwheel                          |
| filevents                            | filevents                         |
| fipp                                 | fipp                              |
| fobos_clj                            | fobos_clj                         |
| fresh                                | fresh                             |
| fs                                   | fs                                |
| fun-utils                            | fun-utils                         |
| garden                               | garden                            |
| gavagai                              | gavagai                           |
| geo-clj                              | geo-clj                           |
| geocoder-clj                         | geocoder-clj                      |
| gloss                                | gloss                             |
| gorilla-renderable                   | gorilla-renderable                |
| grimradical/clj-semver               | clj-semver                        |
| gui-diff                             | gui-diff                          |
| hara                                 | hara                              |
| hdfs-clj                             | hdfs-clj                          |
| hiccup                               | hiccup                            |
| hiccups                              | hiccups                           |
| hickory                              | hickory                           |
| honeysql                             | honeysql                          |
| hornetq-clj/server                   | server                            |
| http-kit                             | http-kit                          |
| http-kit.fake                        | http-kit.fake                     |
| http.async.client                    | http.async.client                 |
| im.chit/hara.class.inheritance       | hara.class.inheritance            |
| im.chit/hara.common.checks           | hara.common.checks                |
| im.chit/hara.common.error            | hara.common.error                 |
| im.chit/hara.common.hash             | hara.common.hash                  |
| im.chit/hara.common.state            | hara.common.state                 |
| im.chit/hara.common.string           | hara.common.string                |
| im.chit/hara.common.watch            | hara.common.watch                 |
| im.chit/hara.data.combine            | hara.data.combine                 |
| im.chit/hara.data.map                | hara.data.map                     |
| im.chit/hara.data.nested             | hara.data.nested                  |
| im.chit/hara.expression.form         | hara.expression.form              |
| im.chit/hara.expression.shorthand    | hara.expression.shorthand         |
| im.chit/hara.function.args           | hara.function.args                |
| im.chit/hara.function.dispatch       | hara.function.dispatch            |
| im.chit/hara.namespace.import        | hara.namespace.import             |
| im.chit/hara.namespace.resolve       | hara.namespace.resolve            |
| im.chit/hara.protocol.state          | hara.protocol.state               |
| im.chit/hara.sort.topological        | hara.sort.topological             |
| im.chit/hara.string.path             | hara.string.path                  |
| im.chit/iroh                         | iroh                              |
| im.chit/purnam.common                | purnam.common                     |
| im.chit/purnam.test                  | purnam.test                       |
| im.chit/ribol                        | ribol                             |
| im.chit/vinyasa.inject               | vinyasa.inject                    |
| im.chit/vinyasa.lein                 | vinyasa.lein                      |
| incise-base-hiccup-layouts           | incise-base-hiccup-layouts        |
| incise-core                          | incise-core                       |
| incise-git-deployer                  | incise-git-deployer               |
| incise-markdown-parser               | incise-markdown-parser            |
| incise-stefon                        | incise-stefon                     |
| inflections                          | inflections                       |
| info.hoetzel/clj-nio2                | clj-nio2                          |
| instaparse                           | instaparse                        |
| interval-metrics                     | interval-metrics                  |
| io                                   | io                                |
| io.aviso/pretty                      | pretty                            |
| io.mandoline/mandoline-core          | mandoline-core                    |
| irclj                                | irclj                             |
| jansi-clj                            | jansi-clj                         |
| jar-migrations                       | jar-migrations                    |
| jarohen/chord                        | chord                             |
| jarohen/nomad                        | nomad                             |
| jayq                                 | jayq                              |
| jeremys/cljss-core                   | cljss-core                        |
| jig/protocols                        | protocols                         |
| jonase/eastwood                      | eastwood                          |
| jonase/kibit                         | kibit                             |
| joplin.core                          | joplin.core                       |
| judgr                                | judgr                             |
| karras                               | karras                            |
| khroma                               | khroma                            |
| kixi/data.vendor.parent              | data.vendor.parent                |
| korma                                | korma                             |
| lamina                               | lamina                            |
| lancet                               | lancet                            |
| lazytest                             | lazytest                          |
| leiningen                            | leiningen                         |
| leiningen-core                       | leiningen-core                    |
| leinjacker                           | leinjacker                        |
| less-awful-ssl                       | less-awful-ssl                    |
| lexington                            | lexington                         |
| lib-noir                             | lib-noir                          |
| liberator                            | liberator                         |
| link                                 | link                              |
| listora/constraint                   | constraint                        |
| longstorm/claude                     | claude                            |
| lonocloud/synthread                  | synthread                         |
| manifold                             | manifold                          |
| marginalia                           | marginalia                        |
| marshmacros                          | marshmacros                       |
| me.raynes/cegdown                    | cegdown                           |
| medley                               | medley                            |
| meridian/shapes                      | shapes                            |
| metosin/ring-http-response           | ring-http-response                |
| metosin/ring-swagger                 | ring-swagger                      |
| metosin/ring-swagger-ui              | ring-swagger-ui                   |
| midje                                | midje                             |
| midje-readme                         | midje-readme                      |
| misaki                               | misaki                            |
| missing-utils                        | missing-utils                     |
| mvxcvi/puget                         | puget                             |
| name.rumford/clojure-carp            | clojure-carp                      |
| native-deps                          | native-deps                       |
| necessary-evil                       | necessary-evil                    |
| net.cgrand/moustache                 | moustache                         |
| net.colourcoding/poppea              | poppea                            |
| net.drib/mrhyde                      | mrhyde                            |
| net.intensivesystems/arrows          | arrows                            |
| net.mikera/core.matrix               | core.matrix                       |
| net.mikera/vectorz-clj               | vectorz-clj                       |
| noencore                             | noencore                          |
| noir                                 | noir                              |
| ns-tracker                           | ns-tracker                        |
| om                                   | om                                |
| onelog                               | onelog                            |
| optimus                              | optimus                           |
| ordered                              | ordered                           |
| ordered-collections                  | ordered-collections               |
| org.blancas/kern                     | kern                              |
| org.blancas/morph                    | morph                             |
| org.bodil/cljs-noderepl              | cljs-noderepl                     |
| org.bodil/lein-noderepl              | lein-noderepl                     |
| org.bodil/redlobster                 | redlobster                        |
| org.clojure/algo.generic             | algo.generic                      |
| org.clojure/algo.monads              | algo.monads                       |
| org.clojure/clojure                  | clojure, clj                      |
| org.clojure/clojurescript            | cljs, clojurescript               |
| org.clojure/core.async               | core.async                        |
| org.clojure/core.cache               | core.cache                        |
| org.clojure/core.contracts           | core.contracts                    |
| org.clojure/core.logic               | core.logic                        |
| org.clojure/core.match               | core.match                        |
| org.clojure/core.memoize             | core.memoize                      |
| org.clojure/core.typed               | core.typed                        |
| org.clojure/core.unify               | core.unify                        |
| org.clojure/data.codec               | data.codec                        |
| org.clojure/data.csv                 | data.csv                          |
| org.clojure/data.finger-tree         | data.finger-tree                  |
| org.clojure/data.fressian            | fressian, data.fressian           |
| org.clojure/data.generators          | data.generators                   |
| org.clojure/data.json                | data.json                         |
| org.clojure/data.priority-map        | data.priority-map, priority-map   |
| org.clojure/data.xml                 | data.xml                          |
| org.clojure/data.zip                 | data.zip                          |
| org.clojure/java.classpath           | java.classpath                    |
| org.clojure/java.data                | java.data                         |
| org.clojure/java.jdbc                | java.jdbc                         |
| org.clojure/java.jmx                 | java.jmx                          |
| org.clojure/jvm.tools.analyzer       | jvm.tools.analyzer                |
| org.clojure/math.combinatorics       | math.combinatorics, combinatorics |
| org.clojure/math.numeric-tower       | numeric-tower, math.numeric-tower |
| org.clojure/test.check               | test.check                        |
| org.clojure/test.generative          | test.generative                   |
| org.clojure/tools.analyzer           | tools.analyzer                    |
| org.clojure/tools.analyzer.jvm       | t.a.jvm, tools.analyzer.jvm       |
| org.clojure/tools.cli                | tools.cli                         |
| org.clojure/tools.macro              | tools.macro                       |
| org.clojure/tools.nrepl              | tools.nrepl                       |
| org.clojure/tools.reader             | tools.reader                      |
| org.clojure/tools.trace              | tools.trace                       |
| org.dthume/data.set                  | data.set                          |
| org.flatland/chronicle               | chronicle                         |
| org.flatland/laminate                | laminate                          |
| org.flatland/telegraph-js            | telegraph-js                      |
| org.flatland/useful                  | useful                            |
| org.immutant/deploy-tools            | deploy-tools                      |
| org.maravillas/ring-core-gae         | ring-core-gae                     |
| org.markdownj/markdownj              | markdownj                         |
| org.ozias.cljlibs/scm                | scm                               |
| org.ozias.cljlibs/shell              | shell                             |
| org.scribe/scribe                    | scribe                            |
| org.thnetos/cd-client                | cd-client                         |
| org.tobereplaced/mapply              | mapply                            |
| oss-jdbc                             | oss-jdbc                          |
| overtone                             | overtone                          |
| pail-cascalog                        | pail-cascalog                     |
| pallet-fsm                           | pallet-fsm                        |
| pallet-map-merge                     | pallet-map-merge                  |
| pallet-thread                        | pallet-thread                     |
| pathetic                             | pathetic                          |
| pedantic                             | pedantic                          |
| perforate                            | perforate                         |
| peridot                              | peridot                           |
| pjstadig/scopes                      | scopes                            |
| polaris                              | polaris                           |
| potemkin                             | potemkin                          |
| primitive-math                       | primitive-math                    |
| prismatic/dommy                      | dommy                             |
| prismatic/fnhouse                    | fnhouse                           |
| prismatic/om-tools                   | om-tools                          |
| prismatic/plumbing                   | plumbing                          |
| prismatic/schema                     | schema                            |
| proteus                              | proteus                           |
| prxml                                | prxml                             |
| puppetlabs/http-client               | http-client                       |
| puppetlabs/kitchensink               | kitchensink                       |
| puppetlabs/trapperkeeper             | trapperkeeper                     |
| purnam/purnam-js                     | purnam-js                         |
| quickie                              | quickie                           |
| quil                                 | quil                              |
| quoin                                | quoin                             |
| radagast                             | radagast                          |
| ragtime                              | ragtime                           |
| re-rand                              | re-rand                           |
| reagent                              | reagent                           |
| reiddraper/simple-check              | simple-check                      |
| reply                                | reply                             |
| retro                                | retro                             |
| rewrite-clj                          | rewrite-clj                       |
| rhizome                              | rhizome                           |
| riddley                              | riddley                           |
| riemann                              | riemann                           |
| riemann-clojure-client               | riemann-clojure-client            |
| ring                                 | ring                              |
| ring-anti-forgery                    | ring-anti-forgery                 |
| ring-cors                            | ring-cors                         |
| ring-middleware-format               | ring-middleware-format            |
| ring-mock                            | ring-mock                         |
| ring-refresh                         | ring-refresh                      |
| ring-reload-modified                 | ring-reload-modified              |
| ring-serve                           | ring-serve                        |
| ring-server                          | ring-server                       |
| ring/ring-codec                      | ring-codec                        |
| ring/ring-core                       | ring-core                         |
| ring/ring-devel                      | ring-devel                        |
| ring/ring-jetty-adapter              | ring-jetty-adapter                |
| ring/ring-json                       | ring-json                         |
| ring/ring-servlet                    | ring-servlet                      |
| ritz/ritz-debugger                   | ritz-debugger                     |
| ritz/ritz-repl-utils                 | ritz-repl-utils                   |
| robert/bruce                         | bruce                             |
| robert/hooke                         | hooke                             |
| ruiyun/tools.timer                   | tools.timer                       |
| sablono                              | sablono                           |
| sass                                 | sass                              |
| schema-contrib                       | schema-contrib                    |
| schematic                            | schematic                         |
| scout                                | scout                             |
| scriptjure                           | scriptjure                        |
| secretary                            | secretary                         |
| seesaw                               | seesaw                            |
| shaky                                | shaky                             |
| shoreleave/shoreleave-browser        | shoreleave-browser                |
| shoreleave/shoreleave-core           | shoreleave-core                   |
| shoreleave/shoreleave-remote         | shoreleave-remote                 |
| shoreleave/shoreleave-remote-ring    | shoreleave-remote-ring            |
| slamhound                            | slamhound                         |
| sligeom                              | sligeom                           |
| slimath                              | slimath                           |
| slingshot                            | slingshot                         |
| sonian/carica                        | carica                            |
| speclj                               | speclj                            |
| spyscope                             | spyscope                          |
| squarepeg                            | squarepeg                         |
| stencil                              | stencil                           |
| storm                                | storm                             |
| substantiation                       | substantiation                    |
| swank-clojure                        | swank-clojure                     |
| swiss-arrows                         | swiss-arrows                      |
| table                                | table                             |
| tailrecursion/cljs-priority-map      | cljs-priority-map                 |
| tailrecursion/cljson                 | cljson                            |
| tentacles                            | tentacles                         |
| test-with-files                      | test-with-files                   |
| test2junit                           | test2junit                        |
| the/parsatron                        | parsatron                         |
| tokyocabinet                         | tokyocabinet                      |
| torpo                                | torpo                             |
| trammel                              | trammel                           |
| transduce                            | transduce                         |
| tron                                 | tron                              |
| up/up-core                           | up-core                           |
| valip                                | valip                             |
| version-clj                          | version-clj                       |
| watchtower                           | watchtower                        |
| weasel                               | weasel                            |
| wrap-js                              | wrap-js                           |
| xrepl                                | xrepl                             |
| zookeeper-clj                        | zookeeper-clj                     |
| zweikopf                             | zweikopf                          |
+--------------------------------------+-----------------------------------+
```

## Possible Issues<a id="sec-1-4" name="sec-1-4"></a>

-   I don't know if relative paths work

### Use with lein-ancient<a id="sec-1-4-1" name="sec-1-4-1"></a>

`lein-plz` uses the same libraries as [lein-ancient](https://github.com/xsc/lein-ancient), the plugin for
upgrading dependencies. It's recommended that users of both
specify the `lein-plz` dependency as follows:

```clojure
[lein-plz "0.2.1" :exclusions [[rewrite-clj] [ancient-clj]]]
```

## License<a id="sec-1-5" name="sec-1-5"></a>

Copyright © 2014 John Walker, [@luxbock (Olli Piepponen)](https://github.com/luxbock) 

Distributed under the Eclipse Public License version 1.0.