import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;
import java.util.concurrent.Semaphore;

/**
 * 
 * I decided to make all the classes for my project in one class,called
 * Atelier.The first variables are globals, the first variable is the seed that
 * I am going to use to generate random numbers,the next one 'marime_fabrica'
 * represents the capacity of the factory which is going to be a number between
 * ten and four,'max' represents the the max value for the elfs,and the max
 * value is the capacity of the factory divided by two,'sd'-I used this variable
 * as an iterator (I am going to explain later why),in 'myGifts' are stored the
 * gifts that are produced by the elfs,'contor_fabrica' represents the number of
 * the factory that is created,In 'oneGift' I extracted the gifts one by one
 * from 'myGifts' and to pass those presents to Ren class,and in 'factoryGifts'
 * i stored the gifts from Ren and took them to MosCraciun class
 *
 */

public class Atelier {

	static Random N = new Random();
	static int marime_Fabrica = N.nextInt(10 - 4 + 1) + 4;
	static int max = marime_Fabrica / 2;
	static int sd = 0;
	static Queue<Integer> myGifts = new LinkedList<Integer>();
	static int contor_Fabrica = 0;
	static Queue<Integer> oneGift = new LinkedList<Integer>();
	static Queue<Integer> factoryGifts = new LinkedList<Integer>();

	static Semaphore semRen = new Semaphore(0);

	/**
	 * 
	 * @param args
	 * @throws InterruptedException
	 * 
	 *             in the main() function I created the threads for the Fabrica
	 *             class.The variable 'nr_Fabrici' represents the number of the
	 *             factories.This number is a random number between five and two.
	 */

	public static void main(String[] args) throws InterruptedException {

		int nr_Fabrici = N.nextInt(5 - 2 + 1) + 2;

		Thread thread[] = new Thread[nr_Fabrici];

		for (int i = 0; i < thread.length; i++) {
			thread[i] = new Fabrica();
		}

		for (int i = 0; i < thread.length; i++) {
			thread[i].start();
			thread[i].sleep(1000);
		}

	}

	/**
	 * 
	 * in the Fabrica class I created the factories,but also the threads for the Ren
	 * class and Elf class.The first variable called 'a' which represents the
	 * factory,and 'elfCurent' represents the curent Elf
	 *
	 */
	public static class Fabrica extends Thread {

		private int[][] a = new int[marime_Fabrica][marime_Fabrica];
		private int elfCurent;

		Ren objRen = new Ren();

		/**
		 * the function createElfs() it`s used to create the Elfs,the maximum number of
		 * the elfs its the length of the factory divided by two,so the number of the
		 * elfs it`s a random number between the maximum number of the elfs and one.
		 */
		private void createElfs() {

			int numar_Elfi = N.nextInt(max - 1 + 1) + 1;
			System.out.println("nr de elfi:" + numar_Elfi);
			Thread thread[] = new Thread[numar_Elfi];

			for (int i = 0; i < thread.length; i++) {
				thread[i] = new Elf(a, numar_Elfi);
			}

			for (int i = 0; i < thread.length; i++) {
				thread[i].start();
				try {
					thread[i].join();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

		}

		int numar_Reni = 8;
		Thread thread[] = new Thread[numar_Reni];

		/**
		 * in the run() function from Fabrica class,first the factories are created,and
		 * all the positions from the factories are initialized with zero.Then the
		 * function createElfs() it`s called,so the elfs will be created,but also they
		 * will start to work in the factories that were created.Then the factory will
		 * ask for the elf`s positions,and then the gifts that were created by the elfs
		 * will be displayed.The gifts are stored in a queue called 'myGifts'.Then the
		 * thread for the Ren class will be created,and these threads will take the
		 * gifts one by one from the queue called 'myGifts' and added to another queue
		 * called 'oneGift',this action will continue until the queue 'myGifts' will be
		 * empty.In the end,the variable 'contor_Fabrica' will be incremented,meaning
		 * that we are done with the current factory.
		 */
		public void run() {

			Elf objElf = new Elf(a, elfCurent);

			System.out.println("Marime Fabrica:" + marime_Fabrica);

			for (int i = 0; i < marime_Fabrica; i++) {
				for (int j = 0; j < marime_Fabrica; j++) {
					a[i][j] = 0;
				}
			}

			createElfs();

			objElf.requestPosition();

			System.out.println("Cadouri aduse in fabrica:" + myGifts);

			for (int i = 0; i < thread.length; i++) {
				thread[i] = new Ren();
			}

			while (myGifts.size() > 0) {

				for (int i = 0; i < thread.length; i++) {

					if (myGifts.size() > 0) {
						oneGift.add(myGifts.poll());
						semRen.release();
						thread[i].start();
						try {
							thread[i].join();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					if (myGifts.size() > 0 && i == thread.length) {
						i = 0;
					} else if (myGifts.size() == 0) {
						i = 7;
					}
				}
			}

			contor_Fabrica++;

		}

		private Queue<Integer> getOneGift() {
			return oneGift;
		}

		private Queue<Integer> getTheGifts() {

			return myGifts;

		}

	}

	/**
	 * 
	 * from the Elf class we are going to let the factory know the location of the
	 * elfs after they are changing the positions,the gifts that were created,and
	 * the number of the current factory.From this class the elfs create the
	 * gifts,so when a gift is created,the elfs are changing the current position.
	 * As variables in this class we have 'a' which represents the factory,I used
	 * the variable 'b' to memorize the old position of the elf,'numar_Elfi' is the
	 * number of the elfs,'contorElf' it`s a iterator for the elfs,and 'numarCadou'
	 * it`s an iterator for the gifts,the variables 'cadouCurent' and 'elfCurent'
	 * represents the current elf and the current gift and 'cont_Fabrica' it`s an
	 * iterator for the factory
	 *
	 */

	public static class Elf extends Thread {

		private int[][] a = new int[marime_Fabrica][marime_Fabrica];
		private int[][] b = new int[marime_Fabrica][marime_Fabrica];
		private int numar_Elfi;
		private int contorElf = -1;
		private int numarCadou = -1;

		private int cadouCurent;
		private int elfCurent;
		private int cont_Fabrica;

		Elf(int[][] a, int numar_Elfi) {
			this.a = a;
			this.numar_Elfi = numar_Elfi;
		}

		/**
		 * 
		 * so with getGift(),the elfs will inform us when they created a gift,the number
		 * of the elf that created the gift,the number of the gift and the number of the
		 * factory that they belong to
		 * 
		 * 
		 * @param i
		 * 
		 *            the first parameter represents the current gift that was created
		 *            by the elfs
		 * 
		 * @param j
		 * 
		 *            the second parameter represents the current elf that created the
		 *            current gift
		 * 
		 * @param k
		 * 
		 *            the third parameter represents the number of the current factory
		 * 
		 * 
		 */

		private void getGift(int i, int j, int k) {
			this.cadouCurent = i;
			this.elfCurent = j;
			this.cont_Fabrica = k;

			System.out.println("Elful cu numarul " + elfCurent + " din fabrica " + cont_Fabrica + " a facut cadoul "
					+ cadouCurent);

		}

		/**
		 * requestPosition() it`s the function used by Fabrica class,in run()
		 * function,with the help of this function the factory will know what`s the
		 * position of the elf in the factory,and the current factory. This function
		 * will we called in the begining when the elfs are stored in the factories and
		 * when the elfs are changing they`re position
		 */

		private void requestPosition() {

			for (int i = 0; i < marime_Fabrica; i++) {
				for (int j = 0; j < marime_Fabrica; j++) {
					if (a[i][j] == 1) {
						contorElf++;
						System.out.println("Elful cu numarul #" + contorElf + " s-a gasit pe linia " + i + " coloana "
								+ j + " din fabrica cu numarul " + contor_Fabrica);
					}
				}
			}

		}

		/**
		 * 
		 * @throws InterruptedException
		 * 
		 * 
		 *             with the help of this function the elfs are changing the
		 *             positions,when they are moving,they are creating a gift. The
		 *             first variable 'nrMutare' can have a value between one and four
		 *             (one-left,two-down,three-right,four-up),so depending on the value
		 *             of this variable,the elf will change his position.'move' it`s
		 *             used as an iterator,it will increment when a elf has moved,if no
		 *             elf has changed his position it will remain on zero.'timeToSleep'
		 *             represents the time to rest for an elf.
		 * 
		 */
		private void changePosition() throws InterruptedException {

			int nrMutare;
			int move = 0;
			int timeToSleep = N.nextInt(50 - 10 + 1) + 10;
			contorElf = -1;

			for (int i = 0; i < marime_Fabrica; i++) {
				for (int j = 0; j < marime_Fabrica; j++) {
					nrMutare = N.nextInt(4 - 1 + 1) + 1;
					if (a[i][j] == 1 && b[i][j] != 1) {
						if (nrMutare == 1 && j == 0) { // perete stanga
							while (nrMutare == 1) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 2 && i == (marime_Fabrica - 1)) { // perete jos
							while (nrMutare == 2) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 3 && j == (marime_Fabrica - 1)) { // perete dreapta
							while (nrMutare == 3) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 4 && i == 0) { // perete sus
							while (nrMutare == 4) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 2 && a[i + 1][j] == 1 && i < (marime_Fabrica - 2)) { // daca are 1 jos
							while (nrMutare == 2) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 3 && a[i][j + 1] == 1 && j < (marime_Fabrica - 2)) { // daca are 1
																								// dreapta
							while (nrMutare == 3) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 4 && a[i - 1][j] == 1 && i > 0) { // daca are 1 sus
							while (nrMutare == 4) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}
						if (nrMutare == 1 && a[i][j - 1] == 1 && j > 0) { // daca are 1 stanga
							while (nrMutare == 1) {
								nrMutare = N.nextInt(4 - 1 + 1) + 1;
							}
						}

						if (nrMutare == 1) {
							a[i][j] = 0;
							a[i][j - 1] = 1;
							b[i][j - 1] = a[i][j - 1];
							move++;
							contorElf++;
							numarCadou++;
							getGift(numarCadou, contorElf, contor_Fabrica); // elfii instiinteaza
							myGifts.add(numarCadou);
						}
						if (nrMutare == 2) {
							a[i][j] = 0;
							a[i + 1][j] = 1;
							b[i + 1][j] = a[i + 1][j];
							move++;
							contorElf++;
							numarCadou++;
							getGift(numarCadou, contorElf, contor_Fabrica); // elfii instiinteaza
							myGifts.add(numarCadou);
						}
						if (nrMutare == 3) {
							a[i][j] = 0;
							a[i][j + 1] = 1;
							b[i][j + 1] = a[i][j + 1];
							move++;
							contorElf++;
							numarCadou++;
							getGift(numarCadou, contorElf, contor_Fabrica); // elfii instiinteaza
							myGifts.add(numarCadou);
						}
						if (nrMutare == 4) {
							a[i][j] = 0;
							a[i - 1][j] = 1;
							b[i - 1][j] = a[i - 1][j];
							move++;
							contorElf++;
							numarCadou++;
							getGift(numarCadou, contorElf, contor_Fabrica); // elfii instiinteaza
							myGifts.add(numarCadou);
						}
					}

					if (move == 0) {
						Thread.sleep(timeToSleep);
					}

				}
			}
		}

		/**
		 * int the run() function from Elf class,we have the variables called 'linie'
		 * and 'coloana' that represent the position of the elfs,the value of those will
		 * be a random number between the capacity of the factory and one. The variable
		 * called 'sd' its a global variable,I used 'sd' as an iterator,it will be
		 * incremented with one after an elf was stored in the factory.When all the elfs
		 * are stored,the value of 'sd' will be equal with the number of the elfs,so the
		 * factory can ask for the position of the elfs,and the elfs can start to create
		 * the gifts,and move in the factory
		 */
		public void run() {

			int linie, coloana;
			int ok = 0;

			sd++;

			while (ok == 0) {

				linie = N.nextInt((marime_Fabrica - 1) - 0 + 1) + 0;
				coloana = N.nextInt((marime_Fabrica - 1) - 0 + 1) + 0;

				if (a[linie][coloana] == 1) {
					ok = 0;
				} else if (a[linie][coloana] == 0) {
					a[linie][coloana] = 1;
					ok = 1;
				}

			}

			if (sd == numar_Elfi) {

				sd = 0;

				requestPosition();

				try {
					changePosition();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

			}

		}

	}

	/**
	 * 
	 * the Ren class receives the gifts from Fabrica class one by one.
	 *
	 */
	public static class Ren extends Thread {

		/**
		 * in the function run() from Ren class,i used a semaphore that will stop the
		 * other Ren threads to acces the critical section,so that with one thread we
		 * can display from Ren the gifts from Fabrica class,and also to display the
		 * gifts that the reindeers took from Fabrica class and stored in the Ren
		 * class.Then the presents from Ren will be stored in another queue called
		 * 'factoryGifts',that is going to go in the MosCraciun class.I also used
		 * another semaphore in the run() function from Fabrica class to give permission
		 * to the Ren threads,after the factory interogates the elfs for they`re
		 * positions and after the elfs inform the factory after they created a
		 * gift,because the Ren threads cannot display the gifts from the Fabrica class
		 * or take a gift before.
		 */

		public void run() {

			Fabrica objFabrica = new Fabrica();
			MosCraciun objMos = new MosCraciun();

			try {
				semRen.acquire();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

			System.out.println("Cadouri afisate din Ren:" + objFabrica.getTheGifts());

			System.out.println("Cadou luat de Reni:" + objFabrica.getOneGift());

			factoryGifts.add(objFabrica.getOneGift().poll());

			objMos.receiveGifts();

		}

		private Queue<Integer> getQueue() {
			return factoryGifts;
		}

	}

	/**
	 * 
	 * in the MosCraciun class it`s the final part of the project,here the gifts
	 * that were bought by the reindeers will be displayed in the function
	 * receiveGifts(),which is called from the Ren class
	 *
	 */
	public static class MosCraciun {

		Ren objRen = new Ren();

		private void receiveGifts() {
			System.out.println("Daruri primite de Mos Craciun:" + objRen.getQueue());
		}

	}

}
