import React, { useState, useEffect, useCallback, useRef } from 'react';
import { Button } from '@/components/ui/button';
import { Progress } from '@/components/ui/progress';
import { Alert, AlertDescription } from '@/components/ui/alert';

const GAME_WIDTH = 800;
const GAME_HEIGHT = 400;
const GRAVITY = 0.5;
const JUMP_STRENGTH = 10;
const MOVE_SPEED = 5;

const EMOJIS = {
  player: '🐢',
  platform: '🟩',
  coin: '💰',
  enemy: '👾',
  heart: '❤️',
};

const EmojiPlatformer = () => {
  const [player, setPlayer] = useState({ x: 50, y: 200, vy: 0 });
  const [platforms, setPlatforms] = useState([]);
  const [coins, setCoins] = useState([]);
  const [enemies, setEnemies] = useState([]);
  const [score, setScore] = useState(0);
  const [lives, setLives] = useState(3);
  const [gameOver, setGameOver] = useState(false);
  const [jumping, setJumping] = useState(false);
  const [movingLeft, setMovingLeft] = useState(false);
  const [movingRight, setMovingRight] = useState(false);
  const [error, setError] = useState(null);

  const gameLoopRef = useRef(null);

  const initGame = useCallback(() => {
    try {
      setPlatforms([
        { x: 0, y: 350, width: 200 },
        { x: 250, y: 300, width: 200 },
        { x: 500, y: 250, width: 200 },
        { x: 750, y: 350, width: 200 },
      ]);
      setCoins([
        { x: 100, y: 300 },
        { x: 300, y: 250 },
        { x: 550, y: 200 },
      ]);
      setEnemies([
        { x: 400, y: 325, direction: 1 },
        { x: 700, y: 325, direction: -1 },
      ]);
      setPlayer({ x: 50, y: 200, vy: 0 });
      setScore(0);
      setLives(3);
      setGameOver(false);
      setError(null);
    } catch (err) {
      console.error("Error initializing game:", err);
      setError("Failed to initialize game. Please try again.");
    }
  }, []);

  useEffect(() => {
    initGame();
  }, [initGame]);

  useEffect(() => {
    if (gameOver || error) {
      if (gameLoopRef.current) {
        clearInterval(gameLoopRef.current);
      }
      return;
    }

    gameLoopRef.current = setInterval(() => {
      try {
        updateGameState();
      } catch (err) {
        console.error("Error in game loop:", err);
        setError("An error occurred during gameplay. Please restart.");
        clearInterval(gameLoopRef.current);
      }
    }, 1000 / 60);  // 60 FPS

    return () => {
      if (gameLoopRef.current) {
        clearInterval(gameLoopRef.current);
      }
    };
  }, [player, platforms, coins, enemies, jumping, movingLeft, movingRight, gameOver, error]);

  const updateGameState = () => {
    // Update player position
    setPlayer(prev => {
      let newX = prev.x;
      if (movingLeft) newX -= MOVE_SPEED;
      if (movingRight) newX += MOVE_SPEED;
      newX = Math.max(0, Math.min(newX, GAME_WIDTH - 20));

      let newY = prev.y + prev.vy;
      let newVy = prev.vy + GRAVITY;

      // Check platform collisions
      const onPlatform = platforms.some(plat => 
        newX < plat.x + plat.width &&
        newX + 20 > plat.x &&
        newY + 20 >= plat.y &&
        newY + 20 <= plat.y + 10
      );

      if (onPlatform) {
        newY = plat.y - 20;
        newVy = 0;
        if (jumping) {
          newVy = -JUMP_STRENGTH;
        }
      }

      // Check boundaries
      if (newY > GAME_HEIGHT - 20) {
        newY = GAME_HEIGHT - 20;
        newVy = 0;
      }

      return { x: newX, y: newY, vy: newVy };
    });

    // Update enemy positions
    setEnemies(prev => prev.map(enemy => {
      let newX = enemy.x + enemy.direction;
      if (newX <= 0 || newX >= GAME_WIDTH - 20) {
        return { ...enemy, x: newX, direction: -enemy.direction };
      }
      return { ...enemy, x: newX };
    }));

    // Check coin collisions
    setCoins(prev => prev.filter(coin => {
      if (Math.abs(coin.x - player.x) < 20 && Math.abs(coin.y - player.y) < 20) {
        setScore(s => s + 10);
        return false;
      }
      return true;
    }));

    // Check enemy collisions
    enemies.forEach(enemy => {
      if (Math.abs(enemy.x - player.x) < 20 && Math.abs(enemy.y - player.y) < 20) {
        setLives(l => {
          if (l > 1) return l - 1;
          setGameOver(true);
          return 0;
        });
      }
    });
  };

  const handleKeyDown = useCallback((e) => {
    if (gameOver || error) return;
    if (e.key === 'ArrowLeft') setMovingLeft(true);
    if (e.key === 'ArrowRight') setMovingRight(true);
    if (e.key === 'ArrowUp') setJumping(true);
  }, [gameOver, error]);

  const handleKeyUp = useCallback((e) => {
    if (e.key === 'ArrowLeft') setMovingLeft(false);
    if (e.key === 'ArrowRight') setMovingRight(false);
    if (e.key === 'ArrowUp') setJumping(false);
  }, []);

  useEffect(() => {
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, [handleKeyDown, handleKeyUp]);

  if (error) {
    return (
      <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-r from-blue-100 to-green-100 p-4">
        <Alert variant="destructive">
          <AlertDescription>{error}</AlertDescription>
        </Alert>
        <Button onClick={initGame} className="mt-4">Restart Game</Button>
      </div>
    );
  }

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-r from-blue-100 to-green-100 p-4">
      <h1 className="text-4xl font-bold mb-4 text-blue-600">Emoji Platformer</h1>
      
      <div className="mb-4">
        Score: {score} | Lives: {lives}
      </div>

      <div 
        style={{
          width: `${GAME_WIDTH}px`,
          height: `${GAME_HEIGHT}px`,
          border: '2px solid black',
          position: 'relative',
          overflow: 'hidden',
        }}
      >
        {platforms.map((plat, index) => (
          <div key={index} style={{
            position: 'absolute',
            left: `${plat.x}px`,
            top: `${plat.y}px`,
            width: `${plat.width}px`,
            height: '20px',
            display: 'flex',
            alignItems: 'center',
          }}>
            {Array(Math.ceil(plat.width / 20)).fill(EMOJIS.platform)}
          </div>
        ))}
        {coins.map((coin, index) => (
          <div key={index} style={{
            position: 'absolute',
            left: `${coin.x}px`,
            top: `${coin.y}px`,
            fontSize: '20px',
          }}>
            {EMOJIS.coin}
          </div>
        ))}
        {enemies.map((enemy, index) => (
          <div key={index} style={{
            position: 'absolute',
            left: `${enemy.x}px`,
            top: `${enemy.y}px`,
            fontSize: '20px',
          }}>
            {EMOJIS.enemy}
          </div>
        ))}
        <div style={{
          position: 'absolute',
          left: `${player.x}px`,
          top: `${player.y}px`,
          fontSize: '20px',
        }}>
          {EMOJIS.player}
        </div>
      </div>

      {gameOver && (
        <div className="mt-4">
          <h2 className="text-2xl font-bold mb-2">Game Over!</h2>
          <Button onClick={initGame}>Rejouer</Button>
        </div>
      )}

      <div className="mt-4 space-x-2">
        <Button onMouseDown={() => setMovingLeft(true)} onMouseUp={() => setMovingLeft(false)} onMouseLeave={() => setMovingLeft(false)} disabled={gameOver}>⬅️</Button>
        <Button onMouseDown={() => setJumping(true)} onMouseUp={() => setJumping(false)} onMouseLeave={() => setJumping(false)} disabled={gameOver}>⬆️</Button>
        <Button onMouseDown={() => setMovingRight(true)} onMouseUp={() => setMovingRight(false)} onMouseLeave={() => setMovingRight(false)} disabled={gameOver}>➡️</Button>
      </div>
    </div>
  );
};

export default EmojiPlatformer;
