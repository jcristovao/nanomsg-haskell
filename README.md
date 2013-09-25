# Overview

This is a Haskell binding for the nanomsg library: <http://nanomsg.org/>.

There's support for [blocking](http://hackage.haskell.org/packages/archive/base/latest/doc/html/Control-Concurrent.html#v:threadWaitRead) send and recv, a non-blocking receive,
and for all the socket types and the functions you need to wire them up and
tear them down again.

Most socket options are available through accessor and mutator
functions. Sockets are typed, transports are not.

# Usage

Simple Pub/sub example:

Server:

    module Main where

    import Nanomsg
    import qualified Data.ByteString.Char8 as C
    import Control.Monad (mapM_)
    import Control.Concurrent (threadDelay)

    main :: IO ()
    main = do
        withSocket Pub $ \s -> do
            _ <- bind s "tcp://*:5560"
            mapM_ (\num -> sendNumber s num) (cycle [1..1000000])
        where
            sendNumber s number = do
                threadDelay 1000        -- let's conserve some cycles
                let numAsString = show number
                send s (C.pack numAsString)

Client:

    module Main where

    import Nanomsg
    import qualified Data.ByteString.Char8 as C
    import Control.Monad (forever)

    main :: IO ()
    main = do
        withSocket Sub $ \s -> do
            _ <- connect s "tcp://localhost:5560"
            subscribe s $ C.pack ""
            forever $ do
                msg <- recv s
                C.putStrLn msg

Nonblocking client:

    module Main where

    import Nanomsg
    import qualified Data.ByteString.Char8 as C
    import Control.Monad (forever)
    import Control.Concurrent (threadDelay)

    main :: IO ()
    main =
        withSocket Sub $ \s -> do
            _ <- connect s "tcp://localhost:5560"
            subscribe s $ C.pack ""
            forever $ do
                threadDelay 700           -- let's conserve some cycles
                msg <- recv' s
                C.putStrLn $ case msg of
                    Nothing -> C.pack "No message"
                    Just s  -> s

