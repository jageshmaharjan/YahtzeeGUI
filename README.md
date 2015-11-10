# YahtzeeGUI
Yahtzee Game GUI Using acm and Yahtzee Library


/*
 * File: Yahtzee.java
 * ------------------
 * This program will eventually play the Yahtzee game.
 */

import java.util.AbstractMap;
import java.util.Arrays;
import java.util.Map.Entry;

import acm.io.IODialog;
import acm.program.GraphicsProgram;
import acm.util.RandomGenerator;

public class Yahtzee extends GraphicsProgram implements YahtzeeConstants
{
	private int[] dice = new int[5];
	private Integer[][] scoreboard = new Integer[N_CATEGORIES][MAX_PLAYERS];

	public static void main(String[] args)
	{
		new Yahtzee().start(args);
	}

	public void run()
	{
		IODialog dialog = getDialog();
		nPlayers = dialog.readInt("Enter number of players");
		playerNames = new String[nPlayers];
		for (int i = 1; i <= nPlayers; i++)
		{
			playerNames[i - 1] = dialog.readLine("Enter name for player " + i);
		}
		display = new YtzDisplay();
		display.init(getGCanvas(), playerNames);
		playGame();
	}

	private void playGame()
	{
		for (int m = 0; m < N_SCORING_CATEGORIES; m++)
		{
			for (int player = 1; player <= nPlayers; player++)
			{
				display.waitForPlayerToClickRoll(player);

				for (int j = 1; j < 6; j++)
				{
					dice[j - 1] = rgen.nextInt(1, 6);
				}
				display.displayDice(dice);

				for (int k = 0; k < 2; k++)
				{
					display.waitForPlayerToSelectDice();
					for (int l = 0; l < 5; l++)
					{
						if (display.isDieSelected(l))
						{
							dice[l] = rgen.nextInt(1, 6);
							break;
						}
					}
					display.displayDice(dice);
				}

				int category = display.waitForPlayerToSelectCategory();

				for (; scoreboard[category - 1][player - 1] != null; category = display.waitForPlayerToSelectCategory())
				{
					display.printMessage("CATEGORY HAS BEEN SELECTED ALREADY");
				}

				int scorepoint = scorecalculate(category);
				scoreboard[category - 1][player - 1] = scorepoint;
				display.updateScorecard(category, player, scorepoint);
			}
		}

		scoreDisplay();
	}

	public void scoreDisplay()
	{
		Entry<String, Integer> max = new AbstractMap.SimpleEntry<>("", 0);

		for (int p = 1; p <= nPlayers; p++)
		{
			scoreboard[UPPER_SCORE - 1][p - 1] = 0;
			scoreboard[LOWER_SCORE - 1][p - 1] = 0;
			scoreboard[TOTAL - 1][p - 1] = 0;

			for (int i = ONES; i < UPPER_SCORE; i++)
			{
				scoreboard[UPPER_SCORE - 1][p - 1] += scoreboard[i - 1][p - 1];
				display.updateScorecard(UPPER_SCORE, p, scoreboard[UPPER_SCORE - 1][p - 1]);
			}

			if (scoreboard[UPPER_SCORE - 1][p - 1] >= 63)
			{
				scoreboard[UPPER_BONUS - 1][p - 1] = 35;
			} else
			{
				scoreboard[UPPER_BONUS - 1][p - 1] = 0;
			}

			display.updateScorecard(UPPER_BONUS, p, scoreboard[UPPER_BONUS - 1][p - 1]);

			for (int i = THREE_OF_A_KIND; i < LOWER_SCORE; i++)
			{
				scoreboard[LOWER_SCORE - 1][p - 1] += scoreboard[i - 1][p - 1];
				display.updateScorecard(LOWER_SCORE, p, scoreboard[LOWER_SCORE - 1][p - 1]);
			}

			scoreboard[TOTAL - 1][p - 1] = scoreboard[UPPER_SCORE - 1][p - 1] + scoreboard[LOWER_SCORE - 1][p - 1];
			display.updateScorecard(TOTAL, p, scoreboard[TOTAL - 1][p - 1]);

			if (scoreboard[TOTAL - 1][p - 1] > max.getValue())
			{
				max = new AbstractMap.SimpleEntry<>(playerNames[p], scoreboard[TOTAL - 1][p - 1]);
			}
		}
		display.printMessage("Winner is " + max.getKey() + " - Score is: " + max.getValue());

	}

	public int scorecalculate(int catagory)
	{
		int scr = 0;
		int a = 0;
		int[] count = getCountOfDice();
		switch (catagory)
		{
		case ONES:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 1)
				{
					a++;
				}
			}
			scr = a;
			break;
		case TWOS:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 2)
				{
					a += 2;
				}
			}
			scr = a;
			break;
		case THREES:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 3)
				{
					a += 3;
				}
			}
			scr = a;
			break;
		case FOURS:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 4)
				{
					a += 4;
				}
			}
			scr = a;
			break;
		case FIVES:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 5)
				{
					a += 5;
				}
			}
			scr = a;
			break;
		case SIXES:
			for (int i = 0; i < N_DICE; i++)
			{
				if (dice[i] == 6)
				{
					a += 6;
				}
			}
			scr = a;
			break;
		case THREE_OF_A_KIND:
			for (int i : count)
			{
				if (i >= 3)
				{
					scr = getSumOfDice();
					break;
				}
			}
			break;
		case FOUR_OF_A_KIND:
			for (int i = 0; i < count.length; i++)
			{
				if (count[i] >= 4)
				{
					scr = getSumOfDice();
					break;
				}
			}
			break;
		case FULL_HOUSE:
			if (Arrays.stream(count).distinct().count() == 2)
			{
				for (int i = 0; i < count.length; i++)
				{
					if (count[i] == 3)
					{
						scr = 25;
					}
				}
			}
			break;
		case SMALL_STRAIGHT:
			if (Arrays.stream(count).distinct().count() == 4)
			{
				if ((count[0] == 0 && count[1] == 0) || (count[5] == 0 && count[4] == 0)
						|| (count[0] == 0 && count[5] == 0))
				{
					scr = 30;
				}
			}
			scr = a;
			break;
		case LARGE_STRAIGHT:
			if (Arrays.stream(count).distinct().count() == 5)
			{
				if ((count[0] == 0) || (count[5] == 0))
				{
					scr = 40;
				}
			}

			break;
		case YAHTZEE:
			if (Arrays.stream(dice).distinct().count() == 1)
				scr = 50;
			break;
		case CHANCE:
			scr = getSumOfDice();
			break;
		default:
		}

		return scr;
	}

	private int[] getCountOfDice()
	{
		int[] count = new int[6];

		for (int i = 0; i < 5; i++)
		{
			count[dice[i] - 1]++;
		}

		return count;
	}

	private int getSumOfDice()
	{
		return Arrays.stream(dice).sum();
	}

	/* Private instance variables */
	private int nPlayers;
	private String[] playerNames;
	private YtzDisplay display;
	private RandomGenerator rgen = new RandomGenerator();
}
