package br.ce.wcaquino.servicos;

import static br.ce.wcaquino.builders.FilmeBuilder.umFilme;
import static br.ce.wcaquino.builders.FilmeBuilder.umFilmeSemEstoque;
import static br.ce.wcaquino.builders.LocacaoBuilder.umLocacao;
import static br.ce.wcaquino.builders.UsuarioBuilder.usiBuilder;
import static br.ce.wcaquino.matchers.MatchersProprios.ehHoje;
import static br.ce.wcaquino.matchers.MatchersProprios.ehHojeComDiferencaDias;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;
import static org.mockito.Mockito.atLeastOnce;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.verifyNoMoreInteractions;
import static org.mockito.Mockito.verifyZeroInteractions;
import static org.mockito.Mockito.when;

import java.util.Arrays;
import java.util.Calendar;
import java.util.Date;
import java.util.List;

import org.junit.Assert;
import org.junit.Assume;
import org.junit.Before;
import  org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ErrorCollector;
import org.junit.rules.ExpectedException;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

import br.ce.wcaquino.daos.LocacaoDAO;
import br.ce.wcaquino.entidades.Filme;
import br.ce.wcaquino.entidades.Locacao;
import br.ce.wcaquino.entidades.Usuario;
import br.ce.wcaquino.exceptions.FilmeSemEstoqueExceptions;
import br.ce.wcaquino.exceptions.LocadoraException;
import br.ce.wcaquino.matchers.MatchersProprios;
import br.ce.wcaquino.utils.DataUtils;
import buildermaster.BuilderMaster;

public class LocalizacaoServiceTeste {
	
	
	//Mocktos--------------------------------------------------
	
	@InjectMocks
	private LocacaoService service;
	
	@Mock
	private LocacaoDAO dao ;
	
	@Mock
	private SPCService spc;
	
	@Mock
	private EmailService email;
	
	//Rules------------------------------------------------------
	@Rule
	public ErrorCollector error = new ErrorCollector();

	@Rule
	public ExpectedException exception = ExpectedException.none();
	
	
	//antes ------------------------------------------------------
	@Before
	public void setup() {
		MockitoAnnotations.initMocks(this);
	}
	
	//Metodos de Teste--------------------------------------------
	@Test
	public void devaAlugarFilme() throws Exception {
		
		Assume.assumeFalse(DataUtils.verificarDiaSemana(new Date(), Calendar.SUNDAY));
		
		// cenario
		Usuario usuario = usiBuilder().agora();
		List<Filme> filmes = Arrays.asList( umFilme().comValor(4.0).agora());

		// acao
		Locacao locacao = service.alugarFilme(usuario, filmes);

		// verificacao
		error.checkThat(locacao.getValor(), is(equalTo(4.0)));
		error.checkThat(DataUtils.isMesmaData(locacao.getDataLocacao(), new Date()), is(true));
		error.checkThat(locacao.getDataLocacao(), ehHoje());
		error.checkThat(DataUtils.isMesmaData(locacao.getDataRetorno(), DataUtils.obterDataComDiferencaDias(1)),is(true));
		error.checkThat(locacao.getDataRetorno(), ehHojeComDiferencaDias(1));
		
		
		
	}

	@Test(expected = FilmeSemEstoqueExceptions.class)
	public void naoDeveAlugarFilmeSemEstoque() throws Exception {
		// cenario
		Usuario usuario = usiBuilder().agora();;
		List<Filme> filmes = Arrays.asList(umFilmeSemEstoque().comValor(4.0).agora());

		// acao
		 service.alugarFilme(usuario, filmes);

	}

	@Test
	public void naoDeveAlugarFilmeSemUsuario() throws FilmeSemEstoqueExceptions {
		// cenario
		List<Filme> filmes = Arrays.asList( umFilme().agora());

		// Acao
		try {
			service.alugarFilme(null, filmes);
			Assert.fail();
		} catch (LocadoraException e) {
			assertThat(e.getMessage(), is("Usuario vazio"));
		}

	}

	@Test
	public void naodeveAlugarFilmeSemFilme() throws FilmeSemEstoqueExceptions, LocadoraException {
		// Cenário
		Usuario usuario = usiBuilder().agora();

		exception.expect(LocadoraException.class);
		exception.expectMessage("Filme vazio");

		// Ação
		service.alugarFilme(usuario, null);

	}
	
	
	@Test 
	public void deveDevolverFilmeNoDomingo () throws FilmeSemEstoqueExceptions, LocadoraException {
	
		Assume.assumeTrue(DataUtils.verificarDiaSemana(new Date(), Calendar.SUNDAY));	
		//cenário
		Usuario usuario = usiBuilder().agora();
		List<Filme> filmes  = Arrays.asList(umFilme().agora());
		
		//Ação
		Locacao retorno =service.alugarFilme(usuario,filmes);
		
		//Verificação
		boolean ehSegunda= DataUtils.verificarDiaSemana(retorno.getDataRetorno(), Calendar.MONDAY);
		Assert.assertTrue(ehSegunda);
		Assert.assertThat(retorno.getDataRetorno(), MatchersProprios.caiEm(Calendar.MONDAY));
		Assert.assertThat(retorno.getDataRetorno(), MatchersProprios.caiNumaSegunda());
		
		
	}
	public static void main (String[] args) {
		new BuilderMaster().gerarCodigoClasse(Locacao.class);
	}
	
	
	
	@Test
	public void naoDeveAlugarFilmeParaNegativadoSPC() throws Exception {
		//cenario
		Usuario usuario = usiBuilder().agora();
		List<Filme> filmes = Arrays.asList(umFilme().agora() );
		
		
		when(spc.possuiNegativacao(Mockito.any(Usuario.class))).thenReturn(true);
		
		
		//Ação 
		try {
			service.alugarFilme(usuario, filmes);
		//Verificação 
			Assert.fail();
		} catch (LocadoraException e) {
			Assert.assertThat(e.getMessage(), is("Usuario Negativado"));
		}
		
		Mockito.verify(spc).possuiNegativacao(usuario);
		
	}
	
	@Test 
	public void deveEnviarEmailParaLocacaoAtrasados(){
		//Cenário 
		Usuario usuario = usiBuilder().agora();
		Usuario usuario2 = usiBuilder().comNome("Usuario 2").agora();
		Usuario usuario3 = usiBuilder().comNome("Usuario 3").agora();

		List<Locacao> locacoes = Arrays.asList(
				umLocacao().comUsuario(usuario).atrasadoBuilder().agora(),
				umLocacao().comUsuario(usuario2).agora(),
				umLocacao().comUsuario(usuario3).atrasadoBuilder().agora(),
				umLocacao().comUsuario(usuario3).atrasadoBuilder().agora());

		when(dao.obterLocacoesPedente()).thenReturn(locacoes);
		//Ação 
		service.notificarAtasos();
		
		//Verificacao 
		verify(email, Mockito.times(3)).notificarAtraso(Mockito.any(Usuario.class));
		verify(email).notificarAtraso(usuario);
		verify(email, atLeastOnce()).notificarAtraso(usuario3);
		verify(email, never()).notificarAtraso(usuario2);
		verifyNoMoreInteractions(email);
		verifyZeroInteractions(spc);

	}
	@Test
	public void  deveTratarErroSPC() throws Exception {
		//Cenário
		Usuario usuario = usiBuilder().agora();
		List<Filme> filmes = Arrays.asList(umFilme().agora());
		
		when(spc.possuiNegativacao(usuario)).thenThrow(new Exception("Falha catastrófica"));
		
		exception.expect(LocadoraException.class);
		exception.expectMessage("Problemas com SPC, tente novamente");
		//exception.expectMessage("Falha catastrófica");

		//Ação 
		service.alugarFilme(usuario, filmes);
		
		//Verificação
	
	
	}
}






 