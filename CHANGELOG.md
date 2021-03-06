# Changelog

## 0.4.0 (2018-09-21)

A major feature release with a significant documentation overhaul and long overdue API cleanup!

This update involves a number of BC breaks due to various changes to make the
API more consistent with the ReactPHP ecosystem. In particular, this now uses
promises consistently as return values instead of accepting callback functions
and this now offers an additional streaming API for processing very large result
sets efficiently.

We realize that the changes listed below may seem a bit overwhelming, but we've
tried to be very clear about any possible BC breaks. See below for changes you
have to take care of when updating from an older version.

*   Feature / BC break: Add Factory to simplify connecting and keeping connection state,
    mark `Connection` class as internal and remove `connect()` method.
    (#64 by @clue)

    ```php
    // old
    $connection = new Connection($loop, $options);
    $connection->connect(function (?Exception $error, $connection) {
        if ($error) {
            // an error occured while trying to connect or authorize client
        } else {
            // client connection established (and authenticated)
        }
    });

    // new
    $factory = new Factory($loop);
    $factory->createConnection($url)->then(
        function (ConnectionInterface $connection) {
            // client connection established (and authenticated)
        },
        function (Exception $e) {
            // an error occured while trying to connect or authorize client
        }
    );
    ```

*   Feature / BC break: Use promises for `query()` method and resolve with `QueryResult` on success and
    and mark all commands as internal and move its base to Commands namespace.
    (#61 and #62 by @clue)

    ```php
    // old
    $connection->query('CREATE TABLE test');
    $connection->query('DELETE FROM user WHERE id < ?', $id);
    $connection->query('SELECT * FROM user', function (QueryCommand $command) {
        if ($command->hasError()) {
            echo 'Error: ' . $command->getError()->getMessage() . PHP_EOL;
        } elseif (isset($command->resultRows)) {
            var_dump($command->resultRows);
        }
    });

    // new
    $connection->query('CREATE TABLE test');
    $connection->query('DELETE FROM user WHERE id < ?', [$id]);
    $connection->query('SELECT * FROM user')->then(function (QueryResult $result) {
        var_dump($result->resultRows);
    }, function (Exception $error) {
        echo 'Error: ' . $error->getMessage() . PHP_EOL;
    });
    ```

*   Feature / BC break: Add new `queryStream()` method to stream result set rows and
    remove undocumented "results" event.
    (#57 and #77 by @clue)

    ```php
    $stream = $connection->queryStream('SELECT * FROM users');

    $stream->on('data', function ($row) {
        var_dump($row);
    });
    $stream->on('end', function () {
        echo 'DONE' . PHP_EOL;
    });
    ```

*   Feature / BC break: Rename `close()` to `quit()`, use promises for `quit()` method and
    add new `close()` method to force-close the connection.
    (#65 and #76 by @clue)

    ```php
    // old: soft-close/quit
    $connection->close(function () {
        echo 'closed';
    });

    // new: soft-close/quit
    $connection->quit()->then(function () {
        echo 'closed';
    });

    // new: force-close
    $connection->close();
    ```

*   Feature / BC break: Use promises for `ping()` method and resolve with void value on success.
    (#63 and #66 by @clue)

    ```php
    // old
    $connection->ping(function ($error, $connection) {
        if ($error) {
            echo 'Error: ' . $error->getMessage() . PHP_EOL;
        } else {
            echo 'OK' . PHP_EOL;
        }
    });

    // new 
    $connection->ping(function () {
        echo 'OK' . PHP_EOL;
    }, function (Exception $error) {
        echo 'Error: ' . $error->getMessage() . PHP_EOL;
    });
    ```

*   Feature / BC break: Define events on ConnectionInterface
    (#78 by @clue)

*   BC break: Remove unneeded `ConnectionInterface` methods `getState()`,
    `getOptions()`, `setOptions()` and `getServerOptions()`, `selectDb()` and `listFields()` dummy.
    (#60 and #68 by @clue)

*   BC break: Mark all protocol logic classes as internal and move to new Io namespace.
    (#53 and #62 by @clue)

*   Fix: Fix executing queued commands in the order they are enqueued
    (#75 by @clue)

*   Fix: Fix reading all incoming response packets until end
    (#59 by @clue)

*   [maintenance] Internal refactoring to simplify connection and authentication logic
    (#69 by @clue)
*   [maintenance] Internal refactoring to remove unneeded references from Commands
    (#67 by @clue)
*   [maintenance] Internal refactoring to remove unneeded EventEmitter implementation and circular references
    (#56 by @clue)
*   [maintenance] Refactor internal parsing logic to separate Buffer class, remove dead code and improve performance
    (#54 by @clue)

## 0.3.3 (2018-06-18)

*   Fix: Reject pending commands if connection is closed
    (#52 by @clue)

*   Fix: Do not support multiple statements for security and API reasons
    (#51 by @clue)

*   Fix: Fix reading empty rows containing only empty string columns 
    (#46 by @clue)

*   Fix: Report correct field length for fields longer than 16k chars
    (#42 by @clue)

*   Add quickstart example and interactive CLI example
    (#45 by @clue)

## 0.3.2 (2018-04-04)

*   Fix: Fix parameter binding if query contains question marks
    (#40 by @clue)

*   Improve test suite by simplifying test structure, improve test isolation and remove dbunit
    (#39 by @clue)

## 0.3.1 (2018-03-26)

*   Feature: Forward compatibility with upcoming ReactPHP components
    (#37 by @clue)

*   Fix: Consistent `connect()` behavior for all connection states
    (#36 by @clue)

*   Fix: Report connection error to `connect()` callback
    (#35 by @clue)

## 0.3.0 (2018-03-13)

*   This is now a community project managed by @friends-of-reactphp. Thanks to
    @bixuehujin for releasing this project under MIT license and handing over!
    (#12 and #33 by @bixuehujin and @clue)

*   Feature / BC break: Update react/socket to v0.8.0
    (#21 by @Fneufneu)

*   Feature: Support passing custom connector and
    load system default DNS config by default
    (#24 by @flow-control and #30 by @clue)

*   Feature: Add `ConnectionInterface` with documentation
    (#26 by @freedemster)

*   Fix: Last query param is lost if no callback is given
    (#22 by @Fneufneu)

*   Fix: Fix memory increase (memory leak due to keeping incoming receive buffer)
    (#17 by @sukui)

*   Improve test suite by adding test instructions and adding Travis CI
    (#34 by @clue and #25 by @freedemster)

*   Improve documentation
    (#8 by @ovr and #10 by @RafaelKa)

## 0.2.0 (2014-10-15)

*   Now compatible with ReactPHP v0.4

## 0.1.0 (2014-02-18)

*   First tagged release (ReactPHP v0.3)
